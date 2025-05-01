# Blockhouse-Trial-Summer25

This project implements a Smart Order Router (SOR) using the static cost model introduced by Cont and Kukanov. The primary goal is to optimize the allocation of a fixed-size market order across multiple venues in order to minimize total execution cost. The model incorporates transaction costs, queue risks, and penalties for over- or under-filling. The backtest simulates how well this routing strategy performs compared to common benchmarks such as Best Ask, TWAP (Time-Weighted Average Price), and VWAP (Volume-Weighted Average Price).

The core of the implementation is the allocate() function, which performs an exhaustive brute-force search to determine the optimal split of the order across available venues at each snapshot. This function calls compute_cost(), which evaluates a candidate allocation using the total cost formula that accounts for venue fees, rebates, execution quantity, and penalties based on three hyperparameters: lambda_over, lambda_under, and theta_queue. The run_backtest() function applies the selected optimal static parameters across the trading day and simulates execution until the entire order is filled.

The script performs a full grid search over combinations of the penalty parameters:

lambda_over: cost for overfilling beyond the order size,

lambda_under: penalty for not filling enough,

theta_queue: cost for queue risk or slippage.

Each parameter is tested over the values [0.01, 0.05, 0.1], resulting in a total of 27 parameter combinations. The best parameter set is selected based on which configuration achieves the lowest total spend while filling the order.

To evaluate the performance of the Smart Order Router, three baselines are implemented. The Best Ask strategy greedily fills from the lowest-priced venue at each snapshot. TWAP evenly splits the order across equally sized time buckets, and VWAP allocates proportionally based on volume. These are compared against the Smart Router in terms of total cash spent and average fill price. A visualization of the average prices and basis point savings relative to the SOR is produced and saved as both results.png and results.pdf.

Results show that the Smart Order Router outperforms both the Best Ask and VWAP baselines, achieving a ~3.6 basis point savings. However, the TWAP strategy showed an abnormally low fill price, indicating a likely bug in the time bucket logic or an issue with early data snapshots. This should be reviewed further before drawing final conclusions.

Several improvements can enhance this system. First, the current brute-force allocator is computationally expensive and does not scale; replacing it with a convex optimization approach (e.g., using cvxpy) would improve efficiency. Second, the TWAP logic should be debugged to ensure accurate temporal alignment. Finally, extending the router to dynamically re-optimize across time instead of using fixed parameters could significantly enhance real-world applicability.

