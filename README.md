Thank you for your patience. I wanted to share a brief update and also clarify the purpose of the optimisation application.

The primary goal of the system is to identify hotspots and inefficiencies in query patterns, and on that front, the application is performing exactly as intended. As mentioned earlier, based on our current understanding, we have been able to identify multiple optimisation opportunities and suggest specific areas where improvements can be made.

That said, to implement the optimal recommendations, we do require functional expertise from your side. Many of these queries are tightly coupled with business logic, and without clarity on their functional intent, there is a risk of altering the behaviour. Our aim is to ensure the optimisation is both technically sound and functionally accurate. This naturally takes time as the team analyses the queries with the limited functional context available.

We truly value the collaborative effort here, and your guidance on the functional side will help us refine and accelerate the optimisation process.

Additionally, we have identified two new query patterns that show significant improvement potential — one with ~70% scan reduction and another with ~100% scan reduction. Both of these queries are executed daily, which means they could deliver meaningful cost benefits once implemented. However, the exact cost impact will be clearer after applying these changes in your DEV or TEST environment, where execution frequency and data volumes reflect your real workload.

Please let us know if we can walk through these findings together or if someone from the functional team can review the logic with us. This will help ensure the recommendations are applied correctly and safely.

Thanks again for the continued collaboration.







Thank you for your detailed inputs. I would like to reiterate that the primary purpose of the application is to identify hotspots in query patterns, and it is performing that job effectively. As shared earlier, based on our initial understanding, we have already identified multiple query patterns with potential optimisation opportunities.

However, as rightly mentioned by your team, functional and business context plays a very critical role in confirming which optimisation is safe to apply without altering the expected behaviour of the queries. This is why the team has been spending time understanding the business logic behind these queries, so that our suggestions are accurate and do not impact the correctness of your data.

In continuation, we have analysed further and are attaching two additional patterns where we see a 70% and 100% scan reduction respectively. These queries are executed daily, so the potential impact is significant. The exact cost savings, however, can be evaluated more accurately once these optimisations are implemented in your DEV or TEST environments and a few cycles of execution are observed.

We also want to highlight that a couple of optimisation patterns were already shared earlier, which also have strong potential for improvement — but again, they need business validation to ensure the intended logic remains intact.

We sincerely appreciate your collaboration so far. With continued functional guidance from your team, we will be able to jointly refine these queries and maximise performance gains without affecting correctness.

Thanks and looking forward to working closely on the next steps.
