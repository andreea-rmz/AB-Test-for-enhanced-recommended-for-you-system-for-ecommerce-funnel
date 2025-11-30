# A/B Test — Recommender System Funnel (EU Region)

1. Project Overview

This project analyzes an A/B test run by an international e-commerce company to evaluate whether a new recommendation-based checkout funnel (Group B) improves user conversion compared to the existing funnel (Group A). The experiment targets new users in the EU region and tracks their behavior for 14 days after registration, focusing on three key events in the funnel: product_page, product_cart and purchase.

The business expectation was ambitious: for Group B to be considered successful, it had to deliver at least a 10% uplift in conversion at each stage of the funnel (product_page → product_cart → purchase) relative to Group A.

2. Objective of the Study

The goal is to determine whether the new recommender-based payment funnel (Group B) leads to a statistically significant and practically meaningful increase in conversion at each stage of the funnel within the first 14 days after signup, compared to the control funnel (Group A). To do this, the analysis combines exploratory data analysis, funnel diagnostics and a two-proportion Z-test for conversion rates.

3. Data Sources

The analysis uses four datasets:

1. A marketing calendar with campaign names, active regions and start/end dates.
2. A user registration table with user_id, signup date (first_date), region anddevice.
3. An event log containing all events (login, product_page, product_cart, purchase, etc.) with timestamps and purchase amounts when applicable.
4. An A/B assignment table indicating which users were assigned to Group A or Group B for the recommender_system_test.

All datasets are loaded, inspected and cleaned in Python using pandas.

4. Data Preparation & Cleaning

-Data preparation focused on making the datasets consistent with the test design:
-Dates were converted to proper datetime types in all relevant tables.
-Regions in the marketing table were split into separate rows so each event-region pair could be analyzed independently.
-The analysis was restricted to users in the EU region who registered between 2020-12-07 and 2020-12-21, in line with the experiment specification.
-Only events occurring within the first 14 days after each user’s registration and before 2021-01-01 were included, ensuring a consistent observation window.
-Users appearing in more than one test group were identified and removed to avoid contamination; after filtering, no overlapping users remained.
-Missing values were minimal and mostly related to purchase amount for non-purchase events, which is expected. No duplicate rows were found.

5. Exploratory Data Analysis (EDA)

-The EDA focused on understanding user behavior along the funnel and assessing the reliability of the event tracking:
-The funnel stages considered were: login → product_page → product_cart → purchase.
-Starting from all users who logged in, the analysis computed how many unique users reached each subsequent stage, as well as conversion relative to the base (logins) and to the previous step.
-The resulting funnel showed a clear drop between login and product_page, and then again between product_page and product_cart, as expected in a typical e-commerce funnel.
-A key anomaly emerged:The number of users who completed a purchase was slightly higher than the number of users who triggered a product_cart event.This suggests that the funnel is not perfectly instrumented: some users may be bypassing the explicit cart step (e.g., using a “quick checkout” option) or certain cart events may be missing from the logs.
-Group A is larger and shows a higher average number of events per user compared to Group B, which is important context when interpreting conversion rates.
-The daily event volume exhibits strong peaks between December 14 and December 21, with a maximum on the 21st, which coincides with the final day of new user enrollment. After that, activity declines, likely driven by seasonality (Christmas and holiday period).
-No users were found assigned to both A and B after cleaning, which supports the internal validity of the experiment.

6. A/B Test — Two-Proportion Z-Test

To formally evaluate the performance of the new funnel, a two-proportion Z-test was used to compare conversion rates between Group A and Group B at each stage of the funnel:
For each event (product_page, product_cart, purchase), the number of unique users who triggered the event was divided by the total number of users in that group to obtain the conversion rate.These rates were compared between A and B using a Z-test for proportions, under the null hypothesis that the conversion rates are equal in both groups.

Key findings from the hypothesis tests:

-At the product_page stage, Group B’s conversion rate was significantly lower than Group A’s, with a p-value close to zero and an estimated decrease of more than 10% in relative terms. This directly contradicts the expected uplift.
-At the product_cart stage, no statistically significant difference was detected between the two groups; their conversion rates can be considered similar in this step.
-At the purchase stage, the test yielded a borderline significant p-value (slightly below 0.05), again indicating a negative effect for Group B: users in B converted to purchases at a lower rate than users in A.
-Overall, instead of achieving a ≥10% uplift, Group B systematically underperforms Group A in the most critical funnel steps.

7. Conclusions & Business Recommendations

The new recommendation-based checkout funnel (Group B) does not improve user conversion in any stage of the funnel. On the contrary, it significantly reduces the likelihood that users will view product pages and complete purchases compared to the existing funnel (Group A).The expected uplift of at least 10% in each stage is not met; in some stages, the effect is negative and statistically significant.
In addition, the funnel instrumentation itself shows inconsistencies, such as more users purchasing than adding items to cart. This suggests that before launching further experiments, the event tracking for key funnel steps (especially product_cart) should be audited and corrected to ensure accurate measurement.

Recommendation:

-Keep the current funnel (Group A) in production and do not roll out Group B.
-Review and fix the event tracking for cart and purchase steps, and only then design and test new recommendation or checkout flows.
