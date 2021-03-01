# B.I.N.O.C.U.L.A.R.S.

This proposal is a work in progress and is meant to evolve in time. We welcome any feedback or comment: please raise new issues should you have any.

---

Relevant Internet advertising is an important component in striking a balance between providing a good end-user experience and allowing publishers and content creators to receive adequate compensation for the content they provide. Hence, the conjunction of user interests (as it is captured by interest groups in TURTLEDOVE) along with relevance with the current context  is a key driver of the open internet advertising ecosystem. Any weakness in leveraging both signals together would undoubtedly hurt both the publisher's revenue and the user experience, exposing them to irrelevant advertisements or worse, unsafe content.

Furthermore, advertisers want to ensure that their products are only appearing on websites that match their brand safety standards and, likewise, publishers want to ensure that all advertisements that appear on their site are suitable for their audience and their own brand image.

Reporting is a key aspect of any full-fledged proposal as the detailed reporting available on the web compared to other channels (billboards, TV ads, etc.) has been one of the main drivers of the advertisers' investment growth in the past decades. Reporting is used for a variety of use-cases ranging from close to real-time campaign management to verifying ex-post that the ad-quality rules have been respected.

Any proposal that does not allow joint use, and reporting of, publisher information along with the interest group is, by design, flawed and will lead to extremely limited adoption from the users, advertisers and publishers. B.I.N.O.C.U.L.A.R.S. - while taking many elements from Reporting in SPARROW - aims at laying out an independent reporting module that could be adopted to "watch" several birds in the aviary/proposals discussed at the W3C, including [FLEDGE](https://github.com/WICG/turtledove/blob/master/FLEDGE.md) as a replacement to the temporary event-level reporting. **While not specifically tied to SPARROW, these measurement reports fit in a privacy-safe ad delivery environment such as described in TURTLEDOVE and [fenced-frames](https://github.com/shivanigithub/fenced-frame)**. Its objective is to prevent any possible association between end-user PII - some being available on publisher side - and interest groups, whilst providing the best possible support for the variety of essential reporting use-cases, including AB testing and lift measurement.

# Table of content
[Three reports](#3-reports-covering-all-usual-measurement-use-cases-while-preserving-full-privacy)

  * [Aggregated report](#aggregated-report)
  * [Report on ads served](#report-on-ads-served)
  * [Ranked-privacy preserving granular report](#ranked-privacy-preserving-granular-report)
  * [Regarding conversion attribution](#regarding-conversion-attribution)

[AB testing and lift measurement](#ab-testing-lift-measurement-and-private-unique-id-count-for-k-anonymity-reporting-computation)

  * [Opportunity-level AB-testing](#opportunity-level-ab-testing)
  * [User-level AB-testing](#user-level-ab-testing)
  * [Lift measurement](#lift-measurement)
  * [Improved k-anonymity computation](#improved-k-anonymity-computation)

[Appendix](#appendix)

# Executive Summary

Reporting relies on a combination of aggregated and [k-anonymous]((https://en.wikipedia.org/wiki/K-anonymity)) granular reports, computed and secured from a user privacy standpoint by the trusted third-party server. There are four different reports:

1. A **near-real-time aggregated report**, with variable latency. This report should be "differentially private".
2. A **delayed served ads report** for publishers. It informs about the creatives and origins of the ads such that the publisher can check if the served ads comply with its ads policy (i.e. publisher ad quality). To meet privacy constraints, a sampling mechanism could be added.
3. A **delayed ranked privacy-preserving granular report based on k-anonymity**:

	- This report is granular, meaning one row per display.
	- A granular report with intentionally bucketed continuous variables and k-anonymity on variables shared by advertiser and publisher (information about the interest group is never available to the publisher, even with appropriate k-anonymity).
	- A different version of this report is available for the advertiser and the publisher.
4. An **aggregated conversion report**, including conversions by users in the control population of the lift AB test (who, by definition didn't see ads) who had opportunities to receive one.

AB testing and lift measurement use a cross-channel framework. This allows for advanced AB testing, essential to all actors across the web, and fair comparisons between marketing vendors/channels, etc. This would bring much-needed clarity and transparency in measuring performance and would be a desirable improvement compared to the current situation.

# 3 reports covering all "usual" measurement use-cases while preserving full privacy

## Aggregated report

This report enables near-real-time management for advertisers and publishers. It relies on differential privacy mechanisms.

Variables such as interest groups, number of displays, number of viewed ads, number of clicks, prices, and "creative IDs" can be exposed here in a privacy-preserving way while keeping the near real-time property which is critical for ad tech players.

However, in order for this report to remain relevant for certain use-cases such as billing, the noise (the differential privacy epsilon) must be restrained. Otherwise, yet another report meeting another set of requirements would be needed for these use-cases.

## Report on ads served

Being able to fully meet the ad quality use case is paramount to avoid not only bad quality ads but also to protect the user from malware advertising. Ad quality is different from many other use cases as it cannot be dealt with an "on average policy". Each case must be handled independently. In order to ensure that all rendered ads were compliant with the publisher ad quality guidelines and, even more important, user safety, the publisher receives **all** web bundles with the click-through URL domain, creative id and the advertiser name. Although the other two types of reports are meant to offer flexibility and completeness while respecting privacy standards, they may not be enough to handle publisher Ad Quality. For this use case, a separate "report on ads served" is necessary.

Besides the complete list, this report does not pass much information and some delay (~24 hours) remains acceptable.

## Ranked-privacy preserving granular report

This report is granular, meaning that each row maps to a display and the label (click/sale) is associated with the exact display for which it occurred. This report can, for example, be used for performance optimization, bug detection on the advertiser side and fraud detection on both sides. While the billing is already covered by the differential private report (Cf. above), this report can help actors get an exact report for this use-case. 

It aims to offer advertisers and publishers a trade-off between the granularity and KPIs accuracy to fulfill their needs while preserving the identity of each user.

To do so, the trusted third party server provides a granular report to the advertiser with k-anonymity on a subset of reported variables (that we call here 'protected variables') with some delay. All the variables that could be used to tie the protected variables together (a.k.a. personal identifiable information (PII)) is obfuscated following the process described below. However, non-protected variables (variables only accessible by one side), such as the click log (is not available in the publisher report), is always recorded at the display level. Therefore, we are preventing any unique join key to appear in the reports to ensure privacy whilst passing as much information as possible to the advertiser and the publisher.

Each actor is limited in the number of queries he can run over each individual raw data point. Without this limit, the ranking of the available dimensions and the K-anonymity wouldn't protect anything. Each display (one row in the raw data) can only be queried once by each actor.

### Non-protected variables

Many variables are relevant or accessible to one player only. For example, ad labels (click ID, opportunity ID, etc.), ABTest ID (see ABTest section below), or the interest group for the advertiser. These variables should be hidden to the other players and do not need other specific attention in this report, they do not represent any risk with regards to the user privacy and can be reported as is at the display granularity.

Some of these variables pose no threat to user privacy and are crucial for advertisers to run performing campaigns. Thus, the ability to get access to them freely (without getting into the granularity vs threshold tradeoff) is extremely beneficial.

### Protected variables

On the other hand, some variables are relevant to both the advertiser and the publisher and represent a privacy risk if exposed as is in the reports. For example:

- Contextual data
- The price of displaying the ad
- The winning DSP / the click-through domain.

Contextual data are very valuable but not all use cases / not all vendors value the same type of contextual data. The report allows the advertiser and the publisher to rank raw contextual variables (e.g. display size) or transformed contextual variables (e.g. truncated URL) by importance according to them, in order to have as much valuable information as possible whilst still preventing colluding publishers and advertisers to join a user with an interest group.

The report relies on k-anonymity, meaning that a feature is available only if at least k similar records (at the UID level) are available (e.g. for a truncated URL to be revealed, you need at least k requests on this truncated URL and the same higher ranked features' modalities). Please refer below for detailed examples of how this system works.

This constraint makes full URLs, with potential PII encoded therein, not usable in such report. The need for providing the transformation to remove these privacy-breaching parts of the URL is thus transferred to the advertiser and publisher (and not the reporter) incentivized by the k-anonymity constraint.

The ranking (provided by the advertiser or publisher) allows the reporter to know which dimension should be sacrificed in case k-anonymity is not respected for a group of reported rows. For k-anonymity to work, continuous variables such as conversion amount (currency) need to be pre-processed either by bucketing them.

### Examples of ranked privacy-preserving report

**The raw data**:

Here we have 9 displays (each with an individual opportunity_ID), to 9 different users, with 4 publisher features, the publisher_UID for each user, the domain, the subdomain, the display size, and a label. The opportunity ID is not shared between the advertiser and publisher.

In this example, we consider **k=2**.

The raw data is the following:

| opportunity_ID | publisher_UID | Domain | Subdomain | Size | Label |
|----------------|---------------|--------|-----------|------|-------|
| abc            | uid1          | A      | A1        | 5    | 0     |
| def            | uid2          | A      | A1        | 10   | 1     |
| ghi            | uid3          | A      | A2        | 10   | 0     |
| jkl            | uid4          | B      | B1        | 5    | 0     |
| mno            | uid5          | B      | B1        | 10   | 1     |
| pqr            | uid6          | B      | B2        | 5    | 0     |
| stu            | uid7          | B      | B2        | 10   | 0     |
| wvx            | uid8          | C      | C1        | 10   | 1     |
| uza            | uid9          | C      | C1        | 10   | 0     |

Now, we show two different reports that could have been requested **by the advertiser**. Note that, as explained above, **the advertiser is only allowed one report, not the two of them**.

#### Example 1: Advertiser dimensions order 1 = [publisher_UID, domain, size, subdomain]

The report passed to the advertiser looks as follow:

| opportunity_ID | Publisher_UID | Domain | Size   | Subdomain | Label |
|----------------|---------------|--------|--------|-----------|-------|
| abc            | Hidden        | A      | Hidden | Hidden    | 0     |
| def            | Hidden        | A      | Hidden | Hidden    | 1     |
| ghi            | Hidden        | A      | Hidden | Hidden    | 0     |
| jkl            | Hidden        | B      | 5      | Hidden    | 0     |
| mno            | Hidden        | B      | 10     | Hidden    | 1     |
| pqr            | Hidden        | B      | 5      | Hidden    | 0     |
| stu            | Hidden        | B      | 10     | Hidden    | 0     |
| wvx            | Hidden        | C      | 10     | C1        | 1     |
| uza            | Hidden        | C      | 10     | C1        | 0     |

The advertiser has access to the opportunity ID as an unprotected variable. The publisher would not have access to it. The publisher would on its side have access to a display ID, again unprotected and only accessible to its side.

Publisher_UIDs are obviously all hidden as there are not k UIDs per UID. Note that this does not stop the algorithm and it returns "Hidden" as a feature value when for instance the value is missing.

All domains are available in clear because there are at least k displays for each domain.

Size: all sizes of domain A are hidden. Why? Because if indeed there are two displays on domain A with size 10, as there is only one remaining display (of size 5), this means that the "Hidden" report size is lower than k, and we must hide the size of displays done on domain A.

Subdomain: all are hidden except subdomains under the C domain, for the same reason as above.

The label is always available because the publisher has no access to the label, and therefore this does not need to be aggregated ("unprotected variable").

#### Example 2: Advertiser dimensions order 2 = [publisher_UID, domain, subdomain, size]

| opportunity_ID | publisher_UID | Domain | Subdomain | Size   | Label |
|----------------|---------------|--------|-----------|--------|-------|
| abc            | Hidden        | A      | Hidden    | Hidden | 0     |
| def            | Hidden        | A      | Hidden    | Hidden | 1     |
| ghi            | Hidden        | A      | Hidden    | Hidden | 0     |
| jkl            | Hidden        | B      | B1        | Hidden | 0     |
| mno            | Hidden        | B      | B1        | Hidden | 1     |
| pqr            | Hidden        | B      | B2        | Hidden | 0     |
| stu            | Hidden        | B      | B2        | Hidden | 0     |
| wvx            | Hidden        | C      | C1        | 10     | 1     |
| uza            | Hidden        | C      | C1        | 10     | 0     |

For similar reasons, a lot of categorical features are put to "Hidden", but here instead of the subdomain being hidden on B, it is the size. This shows the flexibility of this report, allowing the advertiser to choose which features are the most important to them.

### Continuous variables

Some important variables such as prices (in USD, or any currency) are continuous and thus don't fit well in a k-anonymous framework. Indeed, theoretically, the input space being infinite, there could many many cases of only one instance per value. None of these cases would pass the k-anonymity filter, no matter how low k is.

To get access to these variables, we propose to bucketize them. By lowering the input space, we add noise to the initial information but we make sure that multiple rows in the raw data can be considered of the same value and thus pass the k-anonymity filter.

In order to maximize the space reduction while minimizing information loss, the discretization of continuous space can be done intelligently and have a "denser bucketization" for often-used value ranges.

### Value proposition

This report allows for great flexibility, as it empowers actors to rank the features by order of importance according to them. If one advertiser is very interested in geolocation and another does not care, they can rank this dimension differently. The k-anonymity annihilates the incentive to create an individual domain per user and/or to hide PII within a feature since any granular grouping would be hidden in the report.

This also allows for and encourages the advertiser/publisher to find the right tradeoff between latency and precision (the longer the aggregation period is, the more granular the report would be).

Finally, it avoids hiding useful information to the advertiser/publisher that cannot be used to identify users anyway.

## Regarding conversion attribution

Conversion attribution to clicks is not covered by all the previous logs. This is due to the fact that the trusted third-party server does not have access to conversion events. Conversion attribution can be done in BINOCULARS in two ways:
- Link decoration: a click id is added as a decoration to the link, enabling the advertiser to link a subsequent conversion (if any) to the click. Only a click id should be allowed for link decoration, otherwise the k-anonymity of the granular report could be compromised (eg decorating the link with all features asked in  advertiser k-anonymous granular report might reveal hidden features of this particular display).
- Attribution done in browser: similar to [PCM](https://github.com/privacycg/private-click-measurement) or the [lift measurement](#lift-measurement) proposal of this document. The browser would keep a trace of all clicks and conversion, and do attribution. Reporting happens with some random delay after the sale to prevent timing attacks.

An industry-wide consensus has yet to form to decide which of the two solutions is an acceptable trade-off betweeen usability for all parties, user privacy and implicit consent. 

# AB testing, lift measurement and private unique id count for K-anonymity reporting computation

AB testing is a key feature of the measurement on the web and is widely used to compare from different website layouts to complex machine learning algorithms used in AdTech. There are many ways to run an AB test, each serving a slightly different use-case and each bringing its pros and cons. Below are a few different types of AB tests currently used in the AdTech industry with a quick description of the setup, the use-case they are answering and the set of the metrics we consider.

1. [Opportunity Level AB test](#opportunity-level-ab-testing): The decision to apply policy A or policy B happens at each opportunity to show an ad. This can be used, for example, to decide which ad layout or which product is more appealing to users. Slight variants of this setup exist; what matters here is that the population split is not user-centric: a user could see an ad resulting of policy A once and one resulting of policy B at another time.
2. [User Level AB test](#user-level-ab-testing): On the contrary, many AB tests run by Ad tech actors are user-centric: a user should be consistently part of only one population during the duration of the test. This is the case every time you want to test the efficacy of different "marketing messages/strategies" over time.
3. [Incrementality AB test](#lift-measurement): incrementality AB tests (which allow for lift measurement) is a specific case of user-level AB test where you don't treat (show ads) one population in order to compare the performance with the treated population. If you want to know more about incrementality measurement and why we would want to measure incrementally, please refer to this [blog post](https://medium.com/criteo-labs/incrementality-a-simple-question-commanding-subtle-answers-11b68ce344d8) and [this one](https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-conceptual-overview.md) respectively.

## Opportunity Level AB testing

There are 2 major elements necessary to run an AB test:

- the first one is the ability to apply different policies at the opportunity time depending on which population this opportunity falls into.
- the second one is the ability to have a reporting broken down by policy to compare the impacts of different policies.

### At opportunity time

At opportunity time, an opportunity ID can be used by the advertiser to decide what policy to apply for this opportunity and thus have an opportunity level split, as they do today.This opportunity ID is not shared between advertiser and publisher (it could be a fully random Id created by the third party server - or the browser in TURTLEDOVE - not the one used to reconcile bids).

### At reporting time

This opportunity ID is available to the advertiser in the ranked privacy-preserving report described above. Based on this ID available in the report, the advertiser can thus generate a reporting broken down by "policy" since it can know what policy was applied for this particular opportunity ID, hashing it the same way.

Thus, using the reporting already laid out above, the opportunity level AB testing would be fully covered (although you should bear in mind the constraints coming with the ranked privacy-preserving report).

## User-level AB testing

This form of AB testing, that relies on splitting users into several populations and treating each population differently, is at the core of many improvements on the web. As of today, AB testing in web advertising relies mainly on uid hashing that won't be available in the privacy sandbox.

Thus, we must provide with an acceptable privacy-compliant replacement solution that helps make data-driven decisions.

### Our proposal:

- The browser sends at bid time, along with the IG, an *ABTest_ID* between 1 and 16,384 (2^14). This *ABTest_ID* is semi-stable per user (For example, we could imagine that the number for each user would be reset randomly every 2 months).
- The third-party server hashes this *ABTest_ID* differently for each stakeholder (advertisers and publishers), and transform this to an *ABTest_ID_advertiser* between 1 and 32. This ID can be used for AB testing and can be transferred safely ( as an "unprotected variable") in the granular report, as it cannot be used for joining (because the ID for one user is different in the reports produced for the different actors). This enables user-level ABtesting without breaching privacy.

### Value Proposition

The cardinality of the *ABTest_ID_advertiser* is low: this means that the advertiser only has limited options to split the entire pool of users. This also means that an advertiser can run a limited number of tests at the same time.

For example (**In the following example, for the sake of simplicity, the cardinality of the *ABTest_ID_advertiser* is lowered to 10**):

Test 1: 50/50 split

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop |
|----------------------|------------------|------------|
| 0                    | 10%              | Control_1  |
| 1                    | 10%              | Control_1  |
| 2                    | 10%              | Control_1  |
| 3                    | 10%              | Control_1  |
| 4                    | 10%              | Control_1  |
| 5                    | 10%              | Test_1     |
| 6                    | 10%              | Test_1     |
| 7                    | 10%              | Test_1     |
| 8                    | 10%              | Test_1     |
| 9                    | 10%              | Test_1     |

Test 2: 50/50 split

Test 2 split must be orthogonal to Test 1 split for a fair analysis

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop | Test 2 Pop |
|----------------------|------------------|------------|------------|
| 0                    | 10%              | Control_1  | Control_2  |
| 1                    | 10%              | Control_1  | Test_2     |
| 2                    | 10%              | Control_1  | Control_2  |
| 3                    | 10%              | Control_1  | Test_2     |
| 4                    | 10%              | Control_1  | Control_2  |
| 5                    | 10%              | Test_1     | Test_2     |
| 6                    | 10%              | Test_1     | Control_2  |
| 7                    | 10%              | Test_1     | Test_2     |
| 8                    | 10%              | Test_1     | Control_2  |
| 9                    | 10%              | Test_1     | Test_2     |

You can run 2 50/50 split AB tests in parallel. What would happen if the advertiser wanted to run a 90/10 split AB test for the second test (test 2bis)?

Test 2bis: 90/10 split

| *ABTest_ID_advertiser* | Population Share | Test 1 Pop | Test 2bis Pop |
|----------------------|------------------|------------|---------------|
| 0                    | 10%              | Control_1  | ?             |
| 1                    | 10%              | Control_1  | ?             |
| 2                    | 10%              | Control_1  | ?             |
| 3                    | 10%              | Control_1  | ?             |
| 4                    | 10%              | Control_1  | ?             |
| 5                    | 10%              | Test_1     | ?             |
| 6                    | 10%              | Test_1     | ?             |
| 7                    | 10%              | Test_1     | ?             |
| 8                    | 10%              | Test_1     | ?             |
| 9                    | 10%              | Test_1     | ?             |

A 90/10 split, orthogonal to the first test already running is impossible in these conditions. While two 50/50 splits at the same time were possible, a 50/50 and 90/10 were not: tradeoffs exist between the number test you can run at the same time and the population ratios available.

Currently, companies like AdTech providers run tens or hundreds of AB tests simultaneously. This AB test framework would thus be extremely limiting and would require drastic changes in the way they drive their operations. On the other hand, the low cardinality of *ABTest_ID_advertiser* allows it to remain an unprotected variable, meaning that running an AB test won't impact the reporting "quality" and that **advertiser won't be forced to adapt the reporting dimensions depending on the AB tests it is running**.

### Privacy Considerations

As already mentioned, the low cardinality of the *ABTest_ID_advertiser* makes it a limited threat to user privacy despite being treated as an "unprotected variable".

Furthermore, the *ABTest_ID_advertiser* for one user is specific to each advertiser (the same user would have potentially different *ABTest_ID_advertiser* for different advertisers). This means that colluding advertisers would not be able to use this variable to join their reporting.

### Improved K-anonymity computation

This *ABTest_ID* is also used by the third-party server to estimate the number of unique UIDs without having ever access to the UIDs themselves (remember that the third-party server must be able to get the information to apply the K-anonymity filter for the granular reporting). Indeed, we can use the number of distinct ABTest_IDs as a proxy for the number of unique UIDs. For example, if k = 3, the server verifies that there are at least 3 rows for different ABTest_IDs. While it is theoretically possible that two different users have the same *ABTest_ID*, this is unlikely enough (if k remains small and the cardinality of ABTest_ID remains high) so the impact on the report should be very limited.

**This approach means that no unique UIDs are passed to the third-party server**, lowering the amount of trust in them required.  If 2^14 of ABTest_IDs seems too high, consider that with billions of unique UIDs this would reduce the number of UIDs per semi-stable group to hundreds of thousands. These groups would clearly be too large to be a credible threat to user privacy.

## Lift Measurement

Lift measurement requires a form of AB test where a population is not exposed to advertising.

For this part, we built upon the 2 proposals for private lift measurement by Facebook, [here]( https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-first-party.md) and [here]( https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-third-party.md). These APIs are designed to measure the lift of a particular domain and/or a particular ad network. We wish to propose a more flexible framework that would not only answer the use case detailed in these articles but also help advertisers to measure the lift on other dimensions. For example, advertisers must be able to measure the lift brought by different DSPs/ marketing channels and compare them in order to allocate their budget appropriately.

### Initial Proposal

#### Proposal

- Allow a browser to consistently assign users to either a test or control group for an advertiser/redirection domain X a set of dimensions of the choice of the advertiser. Test/Control assignment depends on the redirection domain and a custom hash function.
- Allow the browser to enforce that decision by only showing the ad to people in the test group, suppressing the ad in the control group (show another ad instead).
- The third-party server provides an aggregated reporting, including the sales of the non-treated population.

#### Displaying Ads
- The advertiser (the trusted third party) provides 2 ads: an ad to show to people in the “test group”, and a “fallback ad” to show to people in the control group. The fallback ad can be either a PSA or PSA-like ad or the second choice in the auction).
The first ad contains the ratio *testGroupPercentage* of users in the treated population (who see ads), a hash function and an optional SCOPE, based on the contextual signal.
- The first time the browser needs to make a test/control decision about ads being shown by a given advertiser, it just needs to generate (and store) a random identifier: *advertiser_specific_random_id*. This same identifier will be retrieved by the browser from storage and used on all subsequent test/control decisions made in that browser for ads served by the same advertiser.
- Decision:
	`is_test = IF(contextual_signal IN SCOPE custom_hash_function(advertiser_specific_random_id, IG) % 100 < testGroupPercentage) ELSE TRUE END`
- When such a decision occurs for an advertiser, we say that an opportunity occurred, whether the ad was eventually printed or not.

#### Reporting Conversions

We use here a similar approach to the “Private Click Measurement” and “Conversion Measurement API”.

- The advertiser reports the conversions in a ./well-known location with the various hash_function_id as metadata.
- Browser accesses conversions (./well-known)
- The browser can use the hash functions and the scope in the conversion to get the user's population for each campaign/channel the user was eligible to and sends out a report for each for the conversion to the third party server if the user had one or more opportunities in the defined scope:
	- For each conversion onsite the browser generates N reports for the N hash functions the user had an opportunity for.

**Example**: The user is eligible to 3 IGs, each one with a different split/hash functions (and no defined scope), with the following hash_function_id: “abc”, “def” and “fgh” and converted once on *shop.example*.

He received 2 opportunities for IG with hash_function_id “abc” for which he is in the treatment group (thus 2 displays), 1 opportunity for “def” where is in the control group (thus 0 display) and 0 opportunities for “fgh”.

The generated “reports” would look like this below:

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | abc          |
| isTreated        | 1            |
| Conversion       | 1            |
| Conversion Value | XX           |

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | def          |
| isTreated        | 0            |
| Conversion       | 1            |
| Conversion Value | XX           |

This implies however a change from normal PCM: the browser needs to store more than just clicks, now it must store opportunities to see the ad. We could user something resembling the "trail store" described in [SPURFOWL](https://github.com/AdRoll/privacy/blob/main/SPURFOWL.md), but with the opportunities on top of the other events.

The trusted third-party server aggregates the results and creates a lift aggregated report for each hash function (**With K-anonymity treatment**), adding another report (a fourth one) to the list.

|        Key       |     Value    |
|:----------------:|:------------:|
| reporting-domain | shop.example |
| hash_function_id | abc          |
| isTreated        | 1            |
| Nbr Users        | 33,000       |
| Opportunities    | 100,289      |
| Conversion       | 123          |
| Conversion Value | 1340         |

And a similar report for isTreated = 0.

#### Value Proposition

##### hash function

The *advertiser_specific_random_id* allows the advertiser for the required measurement of the lift brought by its marketing effort as a whole but it is not enough. Indeed, to provide a workable performance framework, the privacy sandbox must allow advertisers to compare the performance of their various campaigns/marketing channels, pilots these channels according to the performance they bring and provide some flexibility in the dimensions you can assess.

Thus we don’t want only to measure the lift/incrementality at the redirection site level (or reporting domain level) but also to measure the impact of each individual campaign/Interest group among other dimensions. The hash function and the scope make sure that the user isn’t consistently assigned to one population or the other for a redirection domain but for a redirection domain X custom dimensions defined by the advertiser using the interest group and the contextual signals available.

Thanks to this custom hash, a user can be in the treatment group for IG_1 of *shop.example* and in the control group for IG_2 of the same site (*shop.example*).

##### Scope

The scope would be useful for an advertiser to test the incrementality of a publisher. Using the contextual signal, the hash function would set the user as part of the test or the control population, but only for the publisher domain, defined as "in scope".

##### Head-to-Head performance comparison

The hash enables an advertiser to compare the performance of two competing marketing channels. For example, *shop.example* wants to compare the lift/incrementality brought by the two remarketers it is working with: AdTech1 and AdTech2.

AdTech1 and AdTech2 would use different functions, thus creating a different split. Each can thus measure the incrementality is bringing to *shop.example* independently of the other.

##### Conversions for the controlled group:

Having access to the conversions in both groups (treatment and control) only for users who had opportunities (displays for one group and "shadow displays" for the other respectively) is extremely beneficial both for minimizing the measurement noise and for the ability to run lift measurement throughout the funnel with a similar framework!

Indeed this allows for measuring the impact of ads taking into accounts only the users who have effectively been impacted (have seen at least one ad in scope).

#### Privacy Considerations

In order to prevent publishers to track and recognize users across sessions even if he deleted his cookies, the *advertiser_specific_random_id*s would be reset as well.

Furthermore, this *advertiser_specific_random_id* presents a high risk of being utilized for cross-site tracking. Even if the identifier itself is not accessible to application code, any kind of leakage of information about this identifier could potentially be used for cross-site tracking.

This implies that it must be **technically impossible** for the publisher to know which ad the person saw (the primary ad or the fallback ad).

The implications of this are discussed [here](https://github.com/w3c/web-advertising/blob/master/private-lift-measurement-third-party.md). While showing ads in a fenced frame and controlling what the publisher can have access to has been discussed and is covered the privacy sandbox, it still raises questions when it comes to pure contextual advertising outside of the privacy sandbox.

Such an attack would require an extreme level of coordination and collusion between the supply side and the advertiser side but would be workable. This needs to be discussed further into details to come up with countermeasures.

#### Other considerations

In order to try to provide consistent test/control status for the same person across multiple devices they own, the browser can sync the *advertiser_specific_random_id* from one device to another.

In order to measure conversion events where the ad-opportunity / ad-impression was on one browser, and the conversion was on another, not only the *advertiser_specific_random_id* but also the locally stored “ad-opportunities requesting attribution” could be synced across devices belonging to the same user.

### Extended Proposal: Have multiple hash functions

Allowing multiple hash functions, each defining a population the user is a part of would enable to have multiple layers of lift measurement running at the same time and thus makes this measurement framework extremely flexible.

For example, if the advertiser passes two hash functions:

- The first layer could allow (for example) for a general lift measurement at the domain level and allow the advertiser to have a broad overview of the performance brought by its marketing efforts.
- The second layer, orthogonal to the first one could allow for campaign level lift measurement and could be used for budget allocation decisions across campaigns and "piloting".

With multiple hash functions, at ad serving time, the decision would be taken by the browser by computing the population for each function and then decide whether the ad should be displayed based on the intersection of the various outputs:

Decision example with 2 functions:

- `isTest1 = IF(contextual_signal IN SCOPE hash_function_1(*advertiser_specific_random_id*, IG) % 100 < *testGroupPercentage*1) ELSE TRUE END`
- `isTest2 = IF(contextual_signal IN SCOPE hash_function_2(*advertiser_specific_random_id*, IG) % 100 < *testGroupPercentage*2) ELSE TRUE END`
- `showAd = isTest1 && isTest2`

The reports including hash_function_id as a dimension, the user and his/her conversions if any would be "included" in the reporting for each hash function.

#### Example

Alice (whom *advertiser_specific_random_id* for *shop.example* is *alice_shop_example_id*) is assigned to one IG (named *IG*) for advertiser *shop.example*. The advertiser wants to measure independently the lift brought by:

1. Its entire marketing effort
2. One publisher: publisher.com

In the web bundle of *IG*, advertiser adds 2 hash functions and 2 related *testGroupPercentage*. Only the second has a scope.

1. `hash_function_1, x, SCOPE_1 = NULL`
2. `hash _function_2, y, SCOPE_2 = publisher.com`

In the example, both functions would be added to all IGs since the measurement the advertiser wishes to run in cross interest-group.

The second "assignment function" displays the ad to all users outside of publisher.com. On publisher.com, the hashing function would allocate the users to the two populations randomly.

Alice visits publisher.com and gets an opportunity for *shop.example* (*shop.example* wins the auction). The browser now gets to decide if it is to print the ad for *shop.example*.

`showAd = ((hash_function_1(alice_shop_example_id) % 100 < x) && (hash_function_2(alice_shop_example_id) % 100 < y)))`

Let's say for the sake of the example that the first term is True but the second term is False. The ad is thus not shown to Alice and another is put in its place.

Alice visits another publisher news.com, which is "out of scope".

`showAd = ((hash_function_1(alice_shop_example_id) % 100 < x) && True = ((hash_function_1(alice_shop_example_id) % 100 < x)`

The first term is the same as on publisher.com (True for alice_shop_example_id). The second term on the other hand is True for all users on this publisher. Thus, this time the ad is shown.

Then Alice comes back to *shop.example* and converts onsite.

The browser generates 2 "reports", sent out to the trusted third party:

|        Key       |      Value      |
|:----------------:|:---------------:|
| reporting-domain | shop.example    |
| hash_function_id | hash_function_1 |
| isTreated        | 1               |
| Conversion       | 1               |
| Conversion Value | X               |

|        Key       |      Value      |
|:----------------:|:---------------:|
| reporting-domain | shop.example    |
| hash_function_id | hash_function_2 |
| isTreated        | 0               |
| Conversion       | 1               |
| Conversion Value | X               |

Had Alice not browsed publisher.com, the second report would not have been generated.

Thus the lift measured thanks to the second hash and scope is the difference in performance between the users who had at least one opportunity on publisher.com and were treated and those who had at least one opportunity on publisher.com and were not treated.

The advertiser receives the aggregated reports:

| reporting-domain | hash_function_id | isTreated | Nbr Users | Opportunities | Conversions | Conversion Value |
|:----------------:|:----------------:|:---------:|:---------:|:-------------:|:-----------:|:----------------:|
| shop.example     | hash_function_1  | 1         | 123,823   | 15,201,456    | 1,456       | 9,463            |
| shop.example     | hash_function_1  | 0         | 123,759   | 15,202,439    | 1,047       | 4,479            |
| shop.example     | hash_function_2  | 1         | 12,436    | 1,302,732     | 145         | 879              |
| shop.example     | hash_function_2  | 0         | 12,435    | 1,302,498     | 142         | 856              |

The advertiser can use these aggregated reports to compute the lift measures it needed.

### Proposal 2

This relies on a completely different idea.
Here, we propose another semi-stable ID, *LIFT_ID* defined by the browser. As for the *ABTest_ID* we could imagine, for example, that the *LIFT_ID*  for each user would be reset randomly every 2-3 months.

The *LIFT_ID* would have a low cardinality (~10) but would be common to all actors (a user would be in the same *LIFT_ID* bucket for all advertisers). For this reason, this variable would be considered protected in the ranked-privacy preserving granular report.

#### Value Proposal

The use of the *LIFT_ID* is quite similar to the *ABTest_ID_advertiser*. The advertiser would use the buckets to treat some of them and not treat the others and measure the difference in performance between the two.

**Pros**:

The buckets being shared by multiple actors, it is convenient to precisely split the population, agnostically of the actor. For example, if *shop.example* want to run a head-to-head comparison of two marketing providers, he can use this ID to split the user pool fairly.

**Cons**:

This method doesn't give access to the pool of users in the control population (untreated) who would have been treated, had they been in the treatment group (users who had an opportunity) and their related onsite sales. The only way to compare both groups is to consider the entire pool of users in the Interest Group. That means that the pool of users is bigger and the results are noisier. This is particularly true for non-retargeting use cases.

In a pure contextual example: users are not known to the advertiser previously to showing an ad.
As the *LIFT_ID* is always available, when a user comes to the site and maybe converts, the advertiser can know if this user belongs to the treatment or the control group. However, there is no way to distinguish users who have been exposed (or at least had an opportunity for users in the control group) from those who have not. Thus, you must include all users in your comparisons.

In a retargeting use case, you can consider in the pool of users only the web users who belong to one of your interest groups, limiting it a lot already but in the contextual use case, you must consider all web users, although you know that you impacted a tiny share of them.

The very fact that the pool of users that take into account is different from one use case to the other skews the performance comparison.

# Appendix

## How does the algorithm used in the ranked privacy-preserving report to obfuscate values work?

The objective of this report is to pass as much information as possible to the advertiser and the publisher without allowing them to associate a user 1st party identity on the publisher side with a user 1st party identity on the advertiser side.

The advertiser ranks protected variables (like contextual data) by the order of importance (by its standards). They can also provide a set of transformations (truncation, bucketing, etc.). The reporting works as follows:

- The reporter (trusted third party server) counts the number of unique "pseudo-UIDs" (this is done in an anonymous way, using the ABTest_IDs) for each modality of the first variable. Please note that this count is done at the publisher domain level, and includes displays that were done on other interest groups and by other DSPs. **This means that one display may be enough for an advertiser to get contextual information, provided that others displays were done by other actors on similar contextual data**:
	
	- For all rows with a variable's modality (e.g. the URL domain of a given website) appearing more than a threshold k in the whole dataset, keep the actual value.
	- For all groups with a number of unique rows lower than k, return "Hidden" instead.
	
		- For this variable, should the number of "Hidden" rows in the group be lower than k, assign to "Hidden" the smallest group with size greater than k, so as to preserve privacy.
		- "Hidden" becomes the new "value" of the variable for the rows of this group, allowing the algorithm to continue if possible and have lower-ranked protected variables not hidden.
- The reporter groups by the n first variables he is asked to group by and verifies if the group still has a number of rows higher than k. Then it provides with the actual value or "Hidden" accordingly.
- Note that this process only takes protected variables into account. Other non-protected dimensions do not go through this procedure and always appear in clear.

The publisher report is produced using the same principle, based on the feature ranking provided by the publisher.
