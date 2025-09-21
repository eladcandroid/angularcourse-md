📚 Library Management System - Design Patterns Exercise
Exercise: Implement Design Patterns in the Library Management System
Building upon your existing Library Management System, implement the three design patterns to enhance the application's architecture and functionality.

📋 Exercise Instructions
1. Book Management Facade Pattern
Create a comprehensive facade service that simplifies all book-related operations:
Requirements:
Combine BookService, InventoryService, and ReservationService
Provide a unified API for book operations
Handle book availability checking across multiple sources
Manage book reservations and waitlists
Cache frequently accessed books
Track reading history


Implementation Tasks:

// Create these methods in BookFacadeService: 
- searchBooks(query: string, filters?: BookFilters): Observable<Book[]> 
- getBookDetails(bookId: string): Observable<BookDetails> 
- reserveBook(bookId: string, memberId: string): Observable<Reservation> 
- checkAvailability(bookId: string): Observable<AvailabilityStatus> 
- getRecommendations(memberId: string): Observable<Book[]> 
- addToWaitlist(bookId: string, memberId: string): Observable<WaitlistEntry>

2. Application State Singleton Pattern
Implement a singleton service to manage global application state:
Requirements:
Library configuration (opening hours, fine rates, loan periods)
User session management
Notification preferences
Theme settings (light/dark mode)
Language preferences
Cache management

// Create LibraryConfigService as singleton: 
- getLibrarySettings(): LibrarySettings 
- updateOpeningHours(hours: OpeningHours): void 
- getFineRate(): number 
- getMaxLoanPeriod(): number 
- getCurrentTheme(): Theme 
- setTheme(theme: Theme): void - clearAllCaches(): void

3. Notification Service Locator Pattern
Build a dynamic notification system that can load different notification providers:
Requirements:
Support multiple notification channels (Email, SMS, Push, In-App)
Dynamic provider loading based on user preferences
Fallback mechanism for failed providers
Template-based notifications
Priority-based routing

// Create notification system with Service Locator: 
- registerProvider(type: string, provider: NotificationProvider): void 
- sendNotification(type: string, recipient: string, template: string, data: any): Observable<boolean> 
- sendBulkNotifications(notifications: BulkNotification[]): Observable<NotificationResult[]> 
- getAvailableProviders(): string[] 
- setUserNotificationPreferences(userId: string, preferences: NotificationPreferences): void

