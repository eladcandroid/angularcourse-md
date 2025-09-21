Practical Exercise
Build a Library Management System with the following structure:

Create 3 Feature Modules:

BooksModule - Book management (display, add, edit, delete)
MembersModule - Member management (member list, member details, borrowing history)
LendingModule - Lending management (new loans, returns, reports)


Requirements for each module:

Separate Routing Module with Lazy Loading
At least 2 components
One service for business logic management
Model/Interface for data types


Create a SharedModule that includes:

Shared Search component
Custom Date Format Pipe
Text Highlight Directive


Create a CoreModule that includes:

AuthService for authentication management
AuthGuard for route protection
HTTP Interceptor for error handling



Bonus: Add State Management with NgRx for each Feature Module separately.