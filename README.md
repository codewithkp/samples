Hi Karthik,

I’d like to share the status of the release deployed yesterday.

The release successfully delivered the planned UI enhancements and support for historical runs, which are now available in production. It also represents the first step towards multi–GCP project support, laying the groundwork for future scalability.

That said, we observed an issue where Version 2 data is not populating in production. This occurred due to a configuration gap introduced during the changes for multi–GCP support, which also impacted yesterday’s batch run. The root cause has been identified, and we are addressing it with a fix scheduled for Friday’s deployment. We are utilizing today for thorough testing to ensure a stable rollout.

We will be ready with the fix by Friday, following thorough testing today. Could you please confirm if deploying this fix on Friday works from your side?
