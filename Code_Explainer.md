# Code Explainer: LLM Server Infrastructure Analytics Pipeline
*This document breaks down the Python code used in the cloud ops monitoring notebook line-by-line for non-technical stakeholders.*

---

## 1. The Server Physics & Simulation Engine
To measure cloud performance without exposing active corporate networks, we used Python to simulate 1,500 continuous Large Language Model API logs.

* **`np.random.exponential(scale=800)`**: This models realistic AI prompt behaviors. Instead of a uniform number, it generates an exponential curve where the vast majority of user prompts are small and fast, while a small minority are massive documents that take significant processing power.
* **`base_latency = 120 + ... + (0.002 * (concurrent_users ** 2))`**: This simulates non-linear cloud mechanics. By squaring the user volume, we programmed a bottleneck rule: traffic under 200 users barely updates server lag, but once traffic spikes past a critical point, the cluster experiences an exponential performance breakdown.
* **`outage_indices` & `latency_ms[outage_indices] = ...`**: To test our detection math, we ordered the engine to pick 15 requests at random and corrupt them with massive 15 to 25-second delay spikes, mimicking a structural database outage or a network router failure.

---

## 2. Statistical Outlier Extraction (The Cleaning Step)
Before analyzing standard server performance, we had to isolate the severe 15-second system outages so they wouldn't skew our standard cluster metrics.

* **`df['Latency_ZScore'] = (df['Latency_MS'] - mean_latency) / std_latency`**: A Z-Score is a statistical measure that tells us how many "steps" away a specific data point is from the typical average baseline. 
* **`df['Latency_ZScore'] > 3`**: Any data point with a Z-Score greater than 3 is mathematically considered a severe anomaly (an extreme outlier). This filter successfully extracted exactly our 15 hidden infrastructure outages, separating them cleanly into their own database slice.

---

## 3. Concurrency Bucket Segmentation
* **`pd.cut(..., bins=bins, labels=labels)`**: This command takes the continuous stream of random user traffic (ranging from 10 to 500 users) and slices it neatly into five labeled corporate operational brackets (Low, Moderate, High, Critical, Breached), allowing us to pinpoint exactly where our infrastructure begins to fall apart.
