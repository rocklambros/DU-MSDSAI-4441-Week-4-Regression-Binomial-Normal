# Problem Set 4 — Explanations

Companion explanations for each question, written for conceptual understanding.

---

## Q1 Part 1: Scatter Plot of GDP 2024 vs 2014

**What it's asking:** Plot each country's 2014 GDP per capita (x-axis) against its 2024 value (y-axis). Each dot = one country or region.

**Why this works:** A "then vs now" scatter is the simplest way to see if wealthy countries stayed wealthy. If every country grew at the same rate, all points would fall on a perfect line. Deviations from that line reveal who grew faster or slower than average.

**What the result tells us:** The strong upward trend means 2014 wealth is a strong predictor of 2024 wealth. The cluster of points at low GDP values (visible thanks to alpha=0.2) shows most countries are bunched at the lower end, with a few rich outliers stretching the axes.

**Gotcha:** The assignment text has a typo — it says "2024 values are in the column labeled X2014" but means X2024. Also, R prefixes numeric column names with `X`, so it's `X2014` not `2014`.

---

## Q1 Part 2: Least-Squares Regression Line

**What it's asking:** Draw the "best fit" straight line through the scatter plot and report its slope and intercept.

**Why this works:** Least-squares finds the line that minimizes the sum of *squared* vertical distances between each point and the line. Squaring means big errors get penalized disproportionately — a point that's 10,000 off contributes 100 million to the total, not just 10,000. This makes the line chase outliers hard.

**What the result tells us:** The slope tells you: for every $1 increase in 2014 GDP per capita, how much did 2024 GDP per capita change on average? The intercept is the predicted 2024 GDP for a hypothetical country with $0 GDP in 2014 (not meaningful literally, but needed for the equation).

**Gotcha:** `lm()` returns named coefficients — `(Intercept)` and `X2014`. The line will be pulled toward Macao, which is a high-leverage outlier.

---

## Q1 Part 3: Total Absolute Error Function

**What it's asking:** Write a function that, for any given line (slope + intercept), computes how far off that line's predictions are from reality — but using absolute values instead of squared values.

**Why this works:** Total absolute error = sum of |actual - predicted| for all points. Unlike squaring, absolute value treats all errors proportionally. A country that's $10,000 off contributes exactly $10,000 to the total, not $100,000,000. This means outliers have less pull.

**What the result tells us:** A single number summarizing how "wrong" the least-squares line is by the absolute-error metric. This gives us a baseline to compare against the optimized line in Part 4.

**Gotcha:** This function is not the one being minimized by `lm()` — least-squares minimizes squared error. So the least-squares line is NOT optimal under this metric.

---

## Q1 Part 4: Minimize Absolute Error with optim()

**What it's asking:** Use R's general-purpose optimizer `optim()` to find a *different* line — one that minimizes total absolute error instead of squared error.

**Why this works:** `optim()` is a numerical search algorithm. You give it a function and starting values, and it tries different inputs until it finds the minimum. We wrap our absolute-error function so `optim()` can search over slope and intercept simultaneously. We use the least-squares estimates as starting values since they should be close.

**What the result tells us:** A slope and intercept that produce a line more robust to outliers than least-squares. The total absolute error should be lower than the LS line's absolute error (since we optimized for it).

**Gotcha:** `optim()` expects a function that takes a single vector parameter, not two separate arguments. That's why we need the wrapper function.

---

## Q1 Part 5: Compare Both Lines

**What it's asking:** Put both lines on the same scatter plot in different colors so you can see how they differ.

**Why this works:** Visual comparison reveals where outliers pull the least-squares line away. The absolute-error line should be closer to the "bulk" of the data, especially in the dense low-GDP region. The zoom windows (0-75K and 75K-175K) make the difference visible.

**What the result tells us:** Macao (highest 2014 GDP, but lower 2024 GDP) pulls the least-squares line's right end downward more than the absolute-error line. In the low-GDP region, the lines may be very similar since most data is there. The lesson: squared error gives outliers outsized influence.

---

## Q2: Identify Regions vs Countries

**What it's asking:** The dataset mixes countries and aggregated regions (like "ASEAN-5", "Advanced Economies"). Find which entries are regions and list them.

**Why this works:** Two clues in the data:
1. Regions have a blank `PRIMARY_DOMESTIC_CURRENCY` — regions don't have a single national currency.
2. Countries appear in 3 rows (one per GDP series: total GDP, per capita domestic, per capita PPP). Regions only have 1 row (just the PPP per capita series).

Using both criteria together prevents false positives — Venezuela also has blank currency (data issues) but has multiple rows, confirming it's a country.

**What the result tells us:** The 11 regional aggregates in the dataset. These are important to know about because including regions alongside countries in regression would double-count GDP (the region totals include their member countries).

---

## Q3 Part 1: Normal Approximation Grid (9 Combinations)

**What it's asking:** For every combination of n in {5, 25, 125} and p in {0.1, 0.2, 0.5}, compare the exact binomial probabilities to the normal approximation using plots and sum-of-absolute-differences.

**Why this works:** The binomial distribution (counting successes in n trials) is discrete — it gives probability for exactly 0, 1, 2, ... successes. The normal distribution is continuous. The "normal approximation to the binomial" is a classic shortcut: instead of computing exact binomial probabilities, approximate them using a normal curve with the same mean (np) and variance (np(1-p)). The continuity correction (using bins like k-0.5 to k+0.5) bridges the discrete-to-continuous gap.

**What the result tells us:** 9 plots showing where the approximation is good (red dots sit on top of blue bars) and where it breaks down (red dots miss). The sum of absolute differences gives a single quality number for each case.

**Gotcha:** We write a reusable function and loop through the grid — this is what "for full credit" requires. The function must both print a plot AND return the numeric error.

---

## Q3 Part 2: Summarize Approximation Quality

**What it's asking:** Look at the 9 results and describe the pattern: when does the normal approximation work well and when does it fail?

**Why this works:** This is about understanding the Central Limit Theorem intuitively:
- **Larger n = better approximation.** More trials → the binomial looks more bell-shaped → the normal fits better.
- **p near 0.5 = better approximation.** When p=0.5, the binomial is perfectly symmetric (like the normal). When p=0.1, it's right-skewed — the normal (symmetric) can't capture that shape well.

**What the result tells us:** The traditional "rule of thumb" — the approximation is decent when both np ≥ 5 and n(1-p) ≥ 5. For example, n=5, p=0.1 gives np=0.5 (way below 5), and the approximation is terrible. But n=125, p=0.5 gives np=62.5, and the approximation is excellent.

**Gotcha:** The summary plot (error vs n, colored by p) makes the two effects visually obvious. All three p-lines decrease with n, but the p=0.1 line starts highest and drops slowest.
