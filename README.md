# A Pet Insurance Pricing Story: The Journey to Profitability

## Introduction: The Curious Start

It all started with a simple, clear goal: build a pricing and reserving model for "Pawtect Insurance," a new pet insurance product. How hard could it be? We had a dataset of 10,000 policies, and the path seemed straightforward. We'd do some EDA, build a smart GLM, and present a shiny, profitable premium structure. We were optimistic and curious.

## Part 1: The Ratemaking Rollercoaster

### The First Sign of Trouble: The "Original" Premium

Our first task was to understand the data. We quickly realized our "original premium" was... well, not very original. It was synthetically generated with a simple formula that was partially tied to the *actual* claims a policy had. This isn't how insurance works; you price on *expected* risk, not what actually happened. This was our first clue that comparing our new model to this baseline would be tricky. It created an artificially low loss ratio of around 34%, a number that would come back to haunt us.

### Building the GLM: A Glimmer of Hope (The "Learning" Phase)

We pushed forward, building a two-part Generalized Linear Model (GLM):
1.  **Frequency Model (Poisson):** To predict how *often* a pet might have a claim.
2.  **Severity Model (Gamma):** To predict how *expensive* a claim might be if it occurs.

The models themselves were sound. They took into account pet age, breed, and location, and the results looked statistically significant. We were on the right track.

### The "Why Is Everything on Fire?" Moment (The "Tired" Phase)

With our models built, we calculated our new "proposed premium." We ran the numbers, and the result was... a **431% loss ratio**.

This was the point where we got tired. A 431% loss ratio doesn't just mean you're not profitable; it means you're paying out over four times what you're taking in. It's a speedrun to insolvency.

### The Investigation (Transparency in Failure)

We dug into the code, line by line. The error was subtle but devastating. Our severity model was trained correctly (only on policies *with* claims), but when we went to predict the severity, we *only predicted it for that same group*. Every policy that didn't have a claim in our dataset was getting a predicted severity of zero, resulting in a pure premium of zero. We had essentially decided that only sick pets should pay for insurance.

### The Fix and a New Mystery (The "Learning" Continues)

The fix was simple: predict the expected severity for *all* policies. Every policy has a *risk* of a claim, so every policy needs a severity prediction. We reran the numbers.

The new loss ratio was **69.98%**. The "Projected Improvement" was **-105.44%**.

We were still losing money, just less spectacularly. How could our "improved" model be more than 100% worse than the original?

### The Epiphany: It's Not the Model, It's the Target

This was the key learning moment. The model was doing *exactly* what we told it to. The problem was in this line of code:
`proposed_premium = pure_premium / (1 - loading_factor)`

With a `loading_factor` of 0.30, we were telling the model to calculate a premium that would result in a **70% loss ratio**. And it did, with near-perfect accuracy (69.98%).

The -105% "improvement" was a red herring. It was a meaningless comparison against our flawed, artificially low "original" loss ratio. We weren't failing to build a good model; we were succeeding at building a model to hit the wrong target.

### Getting it Right: The Final Adjustment

To fix this, we simply had to define our goal correctly. If we wanted to be as profitable as the (artificial) original premium, we needed to target a loss ratio of around 32%. This meant changing our `loading_factor` to 0.68.

With that one change, everything clicked into place. We had a robust, risk-based premium structure that was actually profitable.

## Part 2: The Reserving Side-Quest

While the pricing model was the main adventure, we also tackled reserving:
*   **Chain-Ladder Method:** We used this traditional actuarial method to estimate our ultimate losses and calculate our Incurred But Not Reported (IBNR) reserves.
*   **Model Enhancements:** We even built in functionality to account for veterinary cost inflation, making our model more robust over time.

## Conclusion: A Project's Journey

This project was a journey of curiosity, frustration, and ultimately, learning. It's a transparent look at how a data science project unfolds in the real world. It's not always a straight line. There are dead ends, baffling results, and moments of genuine discovery. By documenting the missteps—the flawed baseline, the logical error in prediction, the misunderstanding of the loading factor—we get a much clearer picture of not just the final result, but the valuable and sometimes tiring process of getting there.
