# Market Basket Analysis — Instacart Dataset

1. **Overview**
This project performs Market Basket Analysis (MBA) using transactional purchase data from the Instacart Online Grocery Shopping Dataset.
The goal is to discover meaningful product associations using:
- Frequent Itemset Mining
- Association Rule Mining
Two algorithms are implemented and compared:
- Apriori
- FP-Growth
These techniques uncover patterns such as frequently co-purchased grocery items, which can support marketing, recommendation, and inventory decisions.

2. **Dataset Description**
**Source**
Instacart Market Basket Dataset (Kaggle).
![dataset](DataPreparation.Dataset.png)

**Files used**

| File                      | Purpose                                     |
|--------------------------|---------------------------------------------|
| order_products__prior.csv | Item-level purchases for previous orders     |
| products.csv             | Product metadata (product_id → product_name) |
| orders.csv               | Order context (order_id, user_id)            |


We only need order-level and product-level data to build transaction baskets.
3. **Key Concepts**

**Transaction**
A single shopping basket (order) containing multiple items purchased together.

**Itemset**
A group of products:
- 1-item set: {Bananas}
- 2-item set: {Bananas, Milk}

**Support**
How often an itemset appears in all orders.
Support(X)=Total Transactions/ Transactions containing X​

**Confidence**
How likely item Y is purchased when X is purchased.
 Support(X)=Total Transactions/Transactions containing X​
 
**Lift**
Indicates whether X and Y occur together more often than random chance.
Support(X)=Total Transactions/Transactions containing X​
 
- Lift > 1: Positive association
- Lift = 1: Independent
- Lift < 1: Negative association

4. **Step-by-Step Methodology**

**Step 1 — Data Preparation**
1. Load transaction table (order_products__prior.csv).
2. Merge with products.csv to obtain product names.
3. Clean strings:
   - lowercasing
   - trimming whitespace
4. Remove:
    - missing names
    - duplicate rows
5. Convert to a basket matrix (order × product), with 1/0 indicating presence.
Because Instacart is large, only the top N most frequent products (e.g., 50) are used to prevent RAM overload.

**Step 2 — Apriori Frequent Itemsets**
- Apriori builds frequent itemsets bottom-up, searching:
1-item sets → 2-item sets → 3-item sets
- Requires repeated scans and candidate generation.
- Computational costs rise dramatically when:
    - support threshold is low
    - dataset is large
    - itemset size increases
To prevent crashes:
- Sampling (e.g., 50,000 orders)
- min_support ≥ 0.02
- max_len ≤ 2
Apriori returns frequent items and co-purchases such as:
```bash
{bananas} , {bag of spinach}, {organic avocado}
```

**Step 3 — FP-Growth Frequent Itemsets**
FP-Growth builds a compressed FP-Tree to avoid candidate generation.

**Advantages:**
- Much faster
- Uses less memory
- Handles large transactional datasets
With identical support thresholds, FP-Growth typically returns nearly the same frequent itemsets as Apriori but in a fraction of the time.

**Step 4 — Association Rules**
Rules are created using mined itemsets.
Example rule:
```bash
{organic strawberries} → {yogurt}
```

With metrics:
- Support: how common the rule is
- Confidence: how often Y occurs given X
- Lift: how much stronger than random chance
Behavior Observed
- Strict thresholds (e.g., confidence ≥ 0.6 + lift ≥ 1.1) → few or zero rules.
- Relaxed thresholds (e.g., confidence ≥ 0.35) → interpretable associations.

**5. Comparative Analysis**

**Performance**
Apriori is slower because it:
- repeatedly scans the dataset
- generates candidate itemsets at every level
FP-Growth:
- compresses transactions into an FP-Tree
- mines frequent patterns without enumeration
- On sampled Instacart data (~50k orders), FP-Growth ran significantly faster.
Output
Both methods returned:
- Nearly identical frequent itemsets
- Similar high-support product pairs

**Rule Generation**
Apriori:
- frequently returns zero rules at strict thresholds
- unstable at low support and large cardinality
FP-Growth:
- stable under lower thresholds
- produces rules more reliably

**6. Practical Takeaways**
- FP-Growth is superior in real business applications.
- Apriori is educational and useful only for small/clean datasets.
- Rule thresholds greatly affect output:
    - Lower = more noise
    - Higher = fewer rules
- Retail baskets are very sparse.
    - Most orders contain 5–10 unique items
    - Only a small fraction of pairs co-occur often

**7. Limitations & Challenges**
- Full Instacart dataset is too large for Apriori in Colab.
- Must sample transactions or increase support.
- Must limit max itemset size to 2–3.
- Association rules can be empty at strict thresholds.
These issues are not “errors” — they are natural consequences of real retail data.

**8. Example Interpretation (for lab write-up)**

“The item pair ‘organic avocado’ + ‘bag of spinach’ appeared together in approximately 2–3% of transactions. Confidence metrics indicated that purchases of organic spinach increase the likelihood of purchasing avocados by a factor of 1.25 (lift > 1), suggesting complementary consumption.”
You can adapt this to your actual output.

**9. Conclusion**
- FP-Growth should be used for analytics on large retail datasets.
- Apriori struggles without sampling and strict constraints.
- Association rule mining reveals co-purchase tendencies that are valuable for:
    - marketing
    - shopping cart recommendations
    - cross-selling strategies
    - inventory placement

**Future Improvements**
- Include department/aisle features.
- Mine temporal sequences (next-purchase prediction).
- Apply collaborative filtering or neural recommenders.
