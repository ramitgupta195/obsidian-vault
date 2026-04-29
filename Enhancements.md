-  When a user’s auth token expires, we should show a clear error message explaining that the session has ended **before** redirecting to the login page. Right now, developers can tell the token expired, but users get no context.

-  Can we introduce more color here? The current UI feels overly formal and flat from a UX perspective.![[Pasted image 20260119103834.png]]

-  In this info section, we’re using a muted grey. Can we use a more noticeable color so these info texts stand out better and visually pop?![[Pasted image 20260119104656.png]]

-  Can we add an **Active / Deactivated** status indicator on these cards?
- The empty state messaging is incorrect.

	- On the **Job Title** page, it shows the empty message for **Department**.
    
	-  On the **Employees** page, it shows the empty message for **Job Title**. 

- We need to implement **auto scroll to the error field** in Add Modules.  
	For example, when adding a Business Unit and a validation error occurs, the view should automatically scroll to the erroneous field. This behavior should be consistent across all Add Modules.
- Toast notifications have inconsistent timing. Sometimes they dismiss automatically, and other times they persist indefinitely.
- Search behavior is inconsistent and doesn’t always return expected results.![[Pasted image 20260119122509.png]]

-  Can we add more color here? Even when **Organization Details** is selected, it still appears grey, which doesn’t clearly indicate selection.![[Pasted image 20260119122712.png]]

-  In Skill Management, while creating a new skill, optional fields appear before required steps. The expected flow should be:  
	**Create Skill Category → Add Inventory**.![[Pasted image 20260119124805.png]]
-  The required step section takes up too much screen space. Instead, can we move these suggestions into an **info (i) icon** that appears on hover?