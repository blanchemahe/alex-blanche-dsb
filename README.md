## Day 1 – Data understanding & target construction

### Dataset exploration

The Kaggle competition provides three files:

- `train.parquet`
- `test.parquet`
- `example_submission.csv`

The `train.parquet` file contains **user activity logs**, not one row per user.  
Each row corresponds to a user interaction with the platform, including:

- `userId`
- `ts` / `time`
- `page`
- `sessionId`
- `location`
- `song`, `artist`
- device and authentication information

There is **no `target` column** in the training data.

### Understanding the churn definition

The competition states that:

> a user churns if they visit the "Cancellation Confirmation" page,
> within 10 days after 2018-11-20.

However, after examining the timestamps, we observed that:

- the last available event is on **2018-11-19 23:58**
- no logs exist after November 20th

Therefore, it is impossible to detect churn *after* the observation window based on timestamps.

### Operational definition of the target

Given the dataset limitations, we adopt the following practical definition:

> `target = 1` if the user has at least one `"Cancellation Confirmation"` event in the logs,
> otherwise `target = 0`.

This approach is consistent with real-world subscription services, where reaching the final cancellation page implies churn.

We constructed the target as follows:

```python
user_target = (
    train.groupby("userId")["page"]
         .apply(lambda pages: int((pages == "Cancellation Confirmation").any()))
         .reset_index()
         .rename(columns={"page": "target"})
)
