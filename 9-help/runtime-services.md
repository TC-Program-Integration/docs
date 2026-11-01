# Runtime services

> **For:** an administrator or a project lead. **Result:** you know where a project describes the access its
> own services need.

This starter has no runtime service of its own: everything it ships runs in the browser, in Dataverse or in
Power Automate, with the signed-in user's access. A project that adds one (an Azure Function, a scheduled job)
replaces this page with its own: what the service is, which identity it runs as, which secrets it keeps and
where, who may rotate them, and how to build the Azure resources.

A project that does so lists this file under `exclude` in its `kit.json`, so `pnpm kit:update` leaves its page
alone, and keeps the page at this path so the link from [access and security](access-and-security.md) still works.
Never put a secret, a token or real data on the page.
