# SQL

# Views vs CTEs

Although there are some differences between them, common table expressions and views seem to perform very similarly. So, when should you use each one?

- **Ad-hoc queries:** For queries that are referenced occasionally (or just once), it’s usually better to use a CTE. If you need the query again, you can just copy the CTE and modify it if necessary.
- **Frequently used queries:** If you tend to reference the same query often, creating a corresponding view is a good idea. However, you’ll need create view permission in your database to create a view.
- **Access management:** A view might be used to restrict particular users’ database access while still allowing them to get the information they need. You can give users access to specific views that query the data they’re allowed to see without exposing the whole database. In such a case, a view provides an additional access layer.
