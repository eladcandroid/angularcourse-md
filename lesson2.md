# Facade Pattern - Topic Overview
The Facade pattern provides a simplified interface to a complex subsystem. It acts as a high-level interface that makes the subsystem easier to use by hiding its complexity. 
In Angular, facades are commonly used with NgRx or complex service interactions to provide a clean API for components.
Benefits:
Simplifies complex APIs
Decouples components from implementation details
Centralizes business logic
Improves testability


Singleton Pattern - Topic Overview
The Singleton pattern ensures a class has only one instance and provides a global point of access to it. In Angular, services provided in 'root' or at module level are singletons by default. This pattern is perfect for managing application-wide state, configuration, or resources.

Benefits:
Ensures single instance across application
Global state management
Resource sharing
Memory efficiency

Service Locator Pattern - Topic Overview
The Service Locator pattern provides a central registry where services can be registered and retrieved. While Dependency Injection is preferred in Angular, Service Locator can be useful for dynamic service resolution or plugin architectures.

⚠️ Note:
This pattern is considered an anti-pattern by some because it creates hidden dependencies. Use it sparingly and prefer Angular's DI when possible.

1. Facade Pattern
   
Description: Simplifies complex subsystem interactions
Example: User management facade that combines API, caching, and validation services
```ts
// Complex subsystems
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of, throwError, BehaviorSubject } from 'rxjs';
import { tap } from 'rxjs/operators';

// User Model
export interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// API Service
@Injectable()
export class UserApiService {
  constructor(private http: HttpClient) {}
  
  getUser(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
  
  updateUser(id: string, data: Partial<User>): Observable<User> {
    return this.http.patch<User>(`/api/users/${id}`, data);
  }
}

// Cache Service
@Injectable()
export class UserCacheService {
  private cache = new Map<string, User>();
  
  set(id: string, user: User): void {
    this.cache.set(id, user);
  }
  
  get(id: string): User | undefined {
    return this.cache.get(id);
  }
  
  clear(): void {
    this.cache.clear();
  }
}

// Validation Service
@Injectable()
export class UserValidationService {
  validateEmail(email: string): boolean {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }
  
  validateAge(age: number): boolean {
    return age >= 18 && age <= 120;
  }
}

// FACADE SERVICE - Simplifies complex subsystem interactions
@Injectable({
  providedIn: 'root'
})
export class UserFacadeService {
  private currentUser$ = new BehaviorSubject<User | null>(null);
  
  constructor(
    private userApi: UserApiService,
    private userCache: UserCacheService,
    private userValidation: UserValidationService
  ) {}
  
  // Simplified interface for components
  getCurrentUser(): Observable<User | null> {
    return this.currentUser$.asObservable();
  }
  
  loadUser(id: string): Observable<User> {
    // Check cache first
    const cachedUser = this.userCache.get(id);
    if (cachedUser) {
      this.currentUser$.next(cachedUser);
      return of(cachedUser);
    }
    
    // Load from API and cache
    return this.userApi.getUser(id).pipe(
      tap(user => {
        this.userCache.set(id, user);
        this.currentUser$.next(user);
      })
    );
  }
  
  updateUserProfile(id: string, email: string, age: number): Observable<User> {
    // Validate
    if (!this.userValidation.validateEmail(email)) {
      return throwError(() => new Error('Invalid email format'));
    }
    
    if (!this.userValidation.validateAge(age)) {
      return throwError(() => new Error('Invalid age'));
    }
    
    // Update
    return this.userApi.updateUser(id, { email, age }).pipe(
      tap(user => {
        this.userCache.set(id, user);
        this.currentUser$.next(user);
      })
    );
  }
  
  logout(): void {
    this.currentUser$.next(null);
    this.userCache.clear();
  }
}// Component usage - clean and simple
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { UserFacadeService, User } from './user-facade.service';

@Component({
  selector: 'app-user-profile',
  template: `
    <div *ngIf="user$ | async as user" class="user-profile">
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
      <p>Age: {{ user.age }}</p>
      
      <div class="actions">
        <button (click)="updateProfile()">Update Profile</button>
        <button (click)="logout()">Logout</button>
      </div>
    </div>
  `,
  styles: [`
    .user-profile {
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    
    .actions {
      margin-top: 20px;
    }
    
    button {
      margin-right: 10px;
      padding: 10px 20px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    button:hover {
      background: #0056b3;
    }
  `]
})
export class UserProfileComponent {
  user$: Observable<User | null> = this.userFacade.getCurrentUser();
  
  constructor(private userFacade: UserFacadeService) {}
  
  ngOnInit(): void {
    // Load user on component init
    this.userFacade.loadUser('123').subscribe();
  }
  
  updateProfile(): void {
    // Simple method call - complexity is hidden in facade
    this.userFacade.updateUserProfile('123', 'newemail@example.com', 25)
      .subscribe(
        user => console.log('Profile updated:', user),
        error => console.error('Update failed:', error)
      );
  }
  
  logout(): void {
    this.userFacade.logout();
    console.log('User logged out');
  }
}
```

2. Singleton Pattern
Description: Configuration and Logger services with single instance guarantee
Example: Application-wide configuration service and centralized logging system

```ts
// Singleton Pattern Example - Configuration and Logger Services
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { APP_INITIALIZER } from '@angular/core';

// Configuration Model
export interface AppConfig {
  apiUrl: string;
  version: string;
  features: {
    enableAnalytics: boolean;
    enableChat: boolean;
    enableDarkMode: boolean;
  };
  environment: 'development' | 'staging' | 'production';
}

// SINGLETON CONFIGURATION SERVICE
// Ensures only one instance exists across the entire application
@Injectable({
  providedIn: 'root'  // This ensures singleton at root level
})
export class ConfigurationService {
  private static instance: ConfigurationService;
  private config: AppConfig;
  private configLoaded = false;
  
  constructor(private http: HttpClient) {
    // Ensure single instance
    if (ConfigurationService.instance) {
      return ConfigurationService.instance;
    }
    ConfigurationService.instance = this;
    console.log('ConfigurationService singleton instance created');
  }
  
  // Load configuration once at app startup
  async loadConfiguration(): Promise<AppConfig> {
    if (this.configLoaded) {
      return this.config;
    }
    
    try {
      this.config = await this.http.get<AppConfig>('/assets/config.json').toPromise();
      this.configLoaded = true;
      console.log('Configuration loaded:', this.config);
      return this.config;
    } catch (error) {
      console.error('Failed to load configuration, using defaults', error);
      this.config = this.getDefaultConfig();
      this.configLoaded = true;
      return this.config;
    }
  }
  
  getConfig(): AppConfig {
    if (!this.configLoaded) {
      throw new Error('Configuration not loaded. Call loadConfiguration() first');
    }
    return this.config;
  }
  
  getValue(key: keyof AppConfig): any {
    return this.config[key];
  }
  
  isFeatureEnabled(feature: keyof AppConfig['features']): boolean {
    return this.config.features[feature];
  }
  
  private getDefaultConfig(): AppConfig {
    return {
      apiUrl: 'http://localhost:3000',
      version: '1.0.0',
      features: {
        enableAnalytics: false,
        enableChat: false,
        enableDarkMode: true
      },
      environment: 'development'
    };
  }
}
// Log Level Enum
export enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3
}

// Log Entry Interface
export interface LogEntry {
  timestamp: Date;
  level: LogLevel;
  message: string;
  context?: any;
  source?: string;
}

// SINGLETON LOGGER SERVICE
// Maintains a single logging instance for the entire application
@Injectable({
  providedIn: 'root'
})
export class LoggerService {
  private logs: LogEntry[] = [];
  private logLevel: LogLevel = LogLevel.INFO;
  private maxLogs = 1000;
  
  private static instance: LoggerService;
  
  constructor(private config: ConfigurationService) {
    if (LoggerService.instance) {
      return LoggerService.instance;
    }
    LoggerService.instance = this;
    console.log('LoggerService singleton instance created');
    
    // Set log level based on environment
    this.initializeLogLevel();
  }
  
  private initializeLogLevel(): void {
    try {
      const appConfig = this.config.getConfig();
      switch (appConfig.environment) {
        case 'development':
          this.logLevel = LogLevel.DEBUG;
          break;
        case 'staging':
          this.logLevel = LogLevel.INFO;
          break;
        case 'production':
          this.logLevel = LogLevel.WARN;
          break;
      }
    } catch {
      // Config not loaded yet, use default
      this.logLevel = LogLevel.INFO;
    }
  }
  
  setLogLevel(level: LogLevel): void {
    this.logLevel = level;
  }
  
  debug(message: string, context?: any, source?: string): void {
    this.log(LogLevel.DEBUG, message, context, source);
  }
  
  info(message: string, context?: any, source?: string): void {
    this.log(LogLevel.INFO, message, context, source);
  }
  
  warn(message: string, context?: any, source?: string): void {
    this.log(LogLevel.WARN, message, context, source);
  }
  
  error(message: string, error?: any, source?: string): void {
    this.log(LogLevel.ERROR, message, error, source);
  }
  
  private log(level: LogLevel, message: string, context?: any, source?: string): void {
    if (level >= this.logLevel) {
      const entry: LogEntry = {
        timestamp: new Date(),
        level,
        message,
        context,
        source: source || this.getCallerSource()
      };
      
      this.logs.push(entry);
      
      // Console output with color coding
      const prefix = `[${LogLevel[level]}] ${entry.timestamp.toISOString()}`;
      const style = this.getConsoleStyle(level);
      
      console.log(`%c${prefix}: ${message}`, style, context || '');
      
      // Maintain max log size
      if (this.logs.length > this.maxLogs) {
        this.logs.shift();
      }
      
      // Send to remote logging in production
      if (level >= LogLevel.ERROR && this.isProduction()) {
        this.sendToRemoteLogging(entry);
      }
    }
  }
  
  private getConsoleStyle(level: LogLevel): string {
    switch (level) {
      case LogLevel.DEBUG: return 'color: gray';
      case LogLevel.INFO: return 'color: blue';
      case LogLevel.WARN: return 'color: orange';
      case LogLevel.ERROR: return 'color: red; font-weight: bold';
      default: return '';
    }
  }
  
  private getCallerSource(): string {
    // Get caller information from stack trace
    const stack = new Error().stack;
    if (stack) {
      const lines = stack.split('\n');
      return lines[3]?.trim() || 'Unknown';
    }
    return 'Unknown';
  }
  
  private isProduction(): boolean {
    try {
      return this.config.getValue('environment') === 'production';
    } catch {
      return false;
    }
  }
  
  private sendToRemoteLogging(entry: LogEntry): void {
    // Implementation for remote logging service
    // This would send logs to services like Sentry, LogRocket, etc.
    console.log('Sending to remote logging:', entry);
  }
  
  getLogs(level?: LogLevel): LogEntry[] {
    if (level !== undefined) {
      return this.logs.filter(log => log.level === level);
    }
    return [...this.logs];
  }
  
  clearLogs(): void {
    this.logs = [];
  }
  
  exportLogs(): string {
    return JSON.stringify(this.logs, null, 2);
  }
}
// APP INITIALIZER - Ensures config is loaded before app starts
export function initializeApp(configService: ConfigurationService): () => Promise<any> {
  return () => configService.loadConfiguration();
}

// In app.module.ts
/*
@NgModule({
  // ...
  providers: [
    {
      provide: APP_INITIALIZER,
      useFactory: initializeApp,
      deps: [ConfigurationService],
      multi: true
    }
  ]
})
export class AppModule { }
*/

// USAGE EXAMPLE IN COMPONENT
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-dashboard',
  template: `
    <div class="dashboard">
      <h1>Dashboard</h1>
      <p>API URL: {{ apiUrl }}</p>
      <p>Version: {{ version }}</p>
      <p>Environment: {{ environment }}</p>
      
      <div class="features">
        <h3>Features:</h3>
        <p>Analytics: {{ features.enableAnalytics ? 'Enabled' : 'Disabled' }}</p>
        <p>Chat: {{ features.enableChat ? 'Enabled' : 'Disabled' }}</p>
        <p>Dark Mode: {{ features.enableDarkMode ? 'Enabled' : 'Disabled' }}</p>
      </div>
      
      <div class="logs">
        <h3>Recent Logs ({{ logCount }})</h3>
        <button (click)="clearLogs()">Clear Logs</button>
        <button (click)="exportLogs()">Export Logs</button>
        <button (click)="testLogging()">Test Logging</button>
      </div>
    </div>
  `
})
export class DashboardComponent implements OnInit {
  apiUrl: string;
  version: string;
  environment: string;
  features: any;
  logCount: number = 0;
  
  constructor(
    private config: ConfigurationService,
    private logger: LoggerService
  ) {
    // Both services are singletons - same instance everywhere
    this.logger.info('DashboardComponent initialized', null, 'DashboardComponent');
  }
  
  ngOnInit(): void {
    // Get configuration values
    const config = this.config.getConfig();
    this.apiUrl = config.apiUrl;
    this.version = config.version;
    this.environment = config.environment;
    this.features = config.features;
    
    // Log component lifecycle
    this.logger.debug('Dashboard loaded with config', config, 'DashboardComponent');
    
    // Update log count
    this.updateLogCount();
  }
  
  testLogging(): void {
    this.logger.debug('Debug message', { test: true });
    this.logger.info('Info message', { timestamp: Date.now() });
    this.logger.warn('Warning message', { alert: 'Check this' });
    this.logger.error('Error message', new Error('Test error'));
    this.updateLogCount();
  }
  
  clearLogs(): void {
    this.logger.clearLogs();
    this.updateLogCount();
  }
  
  exportLogs(): void {
    const logs = this.logger.exportLogs();
    const blob = new Blob([logs], { type: 'application/json' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `logs-${Date.now()}.json`;
    a.click();
  }
  
  private updateLogCount(): void {
    this.logCount = this.logger.getLogs().length;
  }
}
```

1. Service Locator Pattern
URL: https://gist.github.com/eladcandroid/e7f02371bb95a95cbc783e83210e58a0
Description: Dynamic service resolution with plugin system and payment providers
Example: Plugin manager and payment processing system with runtime provider selection

```ts
// Service Locator Pattern Example - Dynamic Service Resolution
import { Injectable, Injector, Type } from '@angular/core';

/**
 * Service Locator Pattern
 * Note: This pattern is sometimes considered an anti-pattern because it creates
 * hidden dependencies. Use it sparingly and prefer Angular's DI when possible.
 * Good use cases: Plugin systems, dynamic service loading, runtime service selection
 */

// SERVICE LOCATOR - Central registry for services
@Injectable({
  providedIn: 'root'
})
export class ServiceLocator {
  private static instance: ServiceLocator;
  private services = new Map<string, any>();
  
  constructor(private injector: Injector) {
    ServiceLocator.instance = this;
    console.log('ServiceLocator initialized');
  }
  
  // Get the singleton instance
  static getInstance(): ServiceLocator {
    if (!ServiceLocator.instance) {
      throw new Error('ServiceLocator not initialized. Ensure it is provided in root module.');
    }
    return ServiceLocator.instance;
  }
  
  // Register a service instance with a token
  register<T>(token: string, service: T): void {
    if (this.services.has(token)) {
      console.warn(`Service ${token} is already registered. Overwriting...`);
    }
    this.services.set(token, service);
    console.log(`Service registered: ${token}`);
  }
  
  // Get a service by token
  get<T>(token: string): T {
    // Check registered services first
    if (this.services.has(token)) {
      return this.services.get(token) as T;
    }
    
    // Try to get from Angular's injector
    try {
      const service = this.injector.get(token as any);
      this.services.set(token, service); // Cache it
      return service;
    } catch (error) {
      throw new Error(`Service ${token} not found in ServiceLocator or Angular DI`);
    }
  }
  
  // Get service by type (class)
  getByType<T>(type: Type<T>): T {
    const token = type.name;
    
    // Check if already registered by type name
    if (this.services.has(token)) {
      return this.services.get(token);
    }
    
    // Try to get from Angular's injector
    try {
      const service = this.injector.get(type);
      this.services.set(token, service); // Cache with type name
      return service;
    } catch (error) {
      throw new Error(`Service ${type.name} not found`);
    }
  }
  
  // Check if service exists
  has(token: string): boolean {
    return this.services.has(token);
  }
  
  // Unregister a service
  unregister(token: string): void {
    if (this.services.has(token)) {
      this.services.delete(token);
      console.log(`Service unregistered: ${token}`);
    }
  }
  
  // Clear all registered services
  clear(): void {
    this.services.clear();
    console.log('All services cleared from ServiceLocator');
  }
  
  // List all registered services
  listServices(): string[] {
    return Array.from(this.services.keys());
  }
}
// PLUGIN SYSTEM USING SERVICE LOCATOR
// Plugin Interface
export interface Plugin {
  name: string;
  version: string;
  description: string;
  initialize(): Promise<void>;
  execute(data: any): any;
  destroy?(): void;
}

// Plugin Manager using Service Locator
@Injectable({
  providedIn: 'root'
})
export class PluginManager {
  private plugins = new Map<string, Plugin>();
  private pluginStates = new Map<string, 'loading' | 'active' | 'error'>();
  
  constructor(private serviceLocator: ServiceLocator) {}
  
  // Register and initialize a plugin
  async registerPlugin(plugin: Plugin): Promise<void> {
    if (this.plugins.has(plugin.name)) {
      throw new Error(`Plugin ${plugin.name} is already registered`);
    }
    
    console.log(`Registering plugin: ${plugin.name} v${plugin.version}`);
    this.pluginStates.set(plugin.name, 'loading');
    
    try {
      await plugin.initialize();
      this.plugins.set(plugin.name, plugin);
      this.serviceLocator.register(`plugin:${plugin.name}`, plugin);
      this.pluginStates.set(plugin.name, 'active');
      console.log(`Plugin activated: ${plugin.name}`);
    } catch (error) {
      this.pluginStates.set(plugin.name, 'error');
      throw new Error(`Failed to initialize plugin ${plugin.name}: ${error}`);
    }
  }
  
  // Get a plugin by name
  getPlugin(name: string): Plugin | undefined {
    return this.serviceLocator.get<Plugin>(`plugin:${name}`);
  }
  
  // Execute a plugin
  async executePlugin(name: string, data: any): Promise<any> {
    const plugin = this.getPlugin(name);
    if (!plugin) {
      throw new Error(`Plugin ${name} not found`);
    }
    
    if (this.pluginStates.get(name) !== 'active') {
      throw new Error(`Plugin ${name} is not active`);
    }
    
    return plugin.execute(data);
  }
  
  // Unregister a plugin
  async unregisterPlugin(name: string): Promise<void> {
    const plugin = this.plugins.get(name);
    if (plugin) {
      if (plugin.destroy) {
        await plugin.destroy();
      }
      this.plugins.delete(name);
      this.serviceLocator.unregister(`plugin:${name}`);
      this.pluginStates.delete(name);
      console.log(`Plugin unregistered: ${name}`);
    }
  }
  
  // List all registered plugins
  listPlugins(): Array<{name: string, version: string, state: string}> {
    return Array.from(this.plugins.entries()).map(([name, plugin]) => ({
      name: plugin.name,
      version: plugin.version,
      state: this.pluginStates.get(name) || 'unknown'
    }));
  }
  
  // Check if plugin exists and is active
  isPluginActive(name: string): boolean {
    return this.pluginStates.get(name) === 'active';
  }
}

// EXAMPLE PLUGINS
// Analytics Plugin
export class AnalyticsPlugin implements Plugin {
  name = 'analytics';
  version = '1.0.0';
  description = 'Tracks user interactions and events';
  private events: any[] = [];
  
  async initialize(): Promise<void> {
    console.log('Analytics plugin initializing...');
    // Load analytics library, configure, etc.
    await this.loadAnalyticsLibrary();
  }
  
  execute(data: { action: string; category: string; label?: string; value?: number }): void {
    const event = {
      timestamp: new Date(),
      ...data
    };
    this.events.push(event);
    console.log('Analytics event tracked:', event);
    
    // Send to analytics service
    this.sendToAnalytics(event);
  }
  
  private async loadAnalyticsLibrary(): Promise<void> {
    // Simulate loading external library
    return new Promise(resolve => setTimeout(resolve, 100));
  }
  
  private sendToAnalytics(event: any): void {
    // Send to Google Analytics, Mixpanel, etc.
    if (window['gtag']) {
      window['gtag']('event', event.action, {
        event_category: event.category,
        event_label: event.label,
        value: event.value
      });
    }
  }
  
  getEvents(): any[] {
    return [...this.events];
  }
  
  destroy(): void {
    this.events = [];
    console.log('Analytics plugin destroyed');
  }
}

// Chat Plugin
export class ChatPlugin implements Plugin {
  name = 'chat';
  version = '2.0.0';
  description = 'Provides live chat functionality';
  private chatWidget: any;
  
  async initialize(): Promise<void> {
    console.log('Chat plugin initializing...');
    await this.loadChatWidget();
  }
  
  execute(data: { action: 'show' | 'hide' | 'message'; payload?: any }): any {
    switch (data.action) {
      case 'show':
        return this.showChat();
      case 'hide':
        return this.hideChat();
      case 'message':
        return this.sendMessage(data.payload);
      default:
        throw new Error(`Unknown chat action: ${data.action}`);
    }
  }
  
  private async loadChatWidget(): Promise<void> {
    // Simulate loading chat widget
    return new Promise(resolve => {
      setTimeout(() => {
        this.chatWidget = { isLoaded: true };
        resolve();
      }, 200);
    });
  }
  
  private showChat(): void {
    console.log('Showing chat widget');
    // Implementation
  }
  
  private hideChat(): void {
    console.log('Hiding chat widget');
    // Implementation
  }
  
  private sendMessage(message: string): void {
    console.log('Sending message:', message);
    // Implementation
  }
  
  destroy(): void {
    this.chatWidget = null;
    console.log('Chat plugin destroyed');
  }
}
// PAYMENT SYSTEM WITH SERVICE LOCATOR
// Payment Provider Interface
export interface PaymentProvider {
  name: string;
  supportedCurrencies: string[];
  processPayment(amount: number, currency: string, details: any): Promise<PaymentResult>;
  validatePaymentDetails(details: any): boolean;
  getTransactionFee(amount: number): number;
}

export interface PaymentResult {
  success: boolean;
  transactionId?: string;
  error?: string;
  timestamp: Date;
}

// Payment Manager using Service Locator for Dynamic Provider Selection
@Injectable({
  providedIn: 'root'
})
export class PaymentManager {
  private providers = new Map<string, PaymentProvider>();
  
  constructor(private serviceLocator: ServiceLocator) {
    this.initializeProviders();
  }
  
  private initializeProviders(): void {
    // Register default payment providers
    this.registerProvider(new PayPalProvider());
    this.registerProvider(new StripeProvider());
    this.registerProvider(new CreditCardProvider());
  }
  
  registerProvider(provider: PaymentProvider): void {
    this.providers.set(provider.name, provider);
    this.serviceLocator.register(`payment:${provider.name}`, provider);
    console.log(`Payment provider registered: ${provider.name}`);
  }
  
  getProvider(name: string): PaymentProvider {
    const provider = this.serviceLocator.get<PaymentProvider>(`payment:${name}`);
    if (!provider) {
      throw new Error(`Payment provider ${name} not found`);
    }
    return provider;
  }
  
  async processPayment(
    providerName: string,
    amount: number,
    currency: string,
    details: any
  ): Promise<PaymentResult> {
    const provider = this.getProvider(providerName);
    
    // Validate payment details
    if (!provider.validatePaymentDetails(details)) {
      return {
        success: false,
        error: 'Invalid payment details',
        timestamp: new Date()
      };
    }
    
    // Check currency support
    if (!provider.supportedCurrencies.includes(currency)) {
      return {
        success: false,
        error: `Currency ${currency} not supported by ${providerName}`,
        timestamp: new Date()
      };
    }
    
    // Process payment
    try {
      const result = await provider.processPayment(amount, currency, details);
      console.log(`Payment processed via ${providerName}:`, result);
      return result;
    } catch (error) {
      return {
        success: false,
        error: error.message,
        timestamp: new Date()
      };
    }
  }
  
  getAvailableProviders(): string[] {
    return Array.from(this.providers.keys());
  }
  
  getProviderFee(providerName: string, amount: number): number {
    const provider = this.getProvider(providerName);
    return provider.getTransactionFee(amount);
  }
}

// PayPal Provider Implementation
class PayPalProvider implements PaymentProvider {
  name = 'paypal';
  supportedCurrencies = ['USD', 'EUR', 'GBP'];
  
  async processPayment(amount: number, currency: string, details: any): Promise<PaymentResult> {
    console.log(`Processing PayPal payment: ${amount} ${currency}`);
    
    // Simulate API call
    await this.simulateApiCall();
    
    return {
      success: true,
      transactionId: `PP-${Date.now()}`,
      timestamp: new Date()
    };
  }
  
  validatePaymentDetails(details: any): boolean {
    return details && details.email && details.password;
  }
  
  getTransactionFee(amount: number): number {
    return amount * 0.029 + 0.30; // 2.9% + $0.30
  }
  
  private simulateApiCall(): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, 1000));
  }
}

// Stripe Provider Implementation
class StripeProvider implements PaymentProvider {
  name = 'stripe';
  supportedCurrencies = ['USD', 'EUR', 'GBP', 'JPY', 'CAD'];
  
  async processPayment(amount: number, currency: string, details: any): Promise<PaymentResult> {
    console.log(`Processing Stripe payment: ${amount} ${currency}`);
    
    await this.simulateApiCall();
    
    return {
      success: true,
      transactionId: `STR-${Date.now()}`,
      timestamp: new Date()
    };
  }
  
  validatePaymentDetails(details: any): boolean {
    return details && details.cardNumber && details.cvv && details.expiryDate;
  }
  
  getTransactionFee(amount: number): number {
    return amount * 0.027 + 0.25; // 2.7% + $0.25
  }
  
  private simulateApiCall(): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, 800));
  }
}

// Credit Card Provider Implementation
class CreditCardProvider implements PaymentProvider {
  name = 'creditcard';
  supportedCurrencies = ['USD', 'EUR'];
  
  async processPayment(amount: number, currency: string, details: any): Promise<PaymentResult> {
    console.log(`Processing Credit Card payment: ${amount} ${currency}`);
    
    await this.simulateApiCall();
    
    return {
      success: true,
      transactionId: `CC-${Date.now()}`,
      timestamp: new Date()
    };
  }
  
  validatePaymentDetails(details: any): boolean {
    return details && 
           details.cardNumber && 
           details.cardholderName && 
           details.cvv && 
           details.expiryMonth && 
           details.expiryYear;
  }
  
  getTransactionFee(amount: number): number {
    return amount * 0.025 + 0.10; // 2.5% + $0.10
  }
  
  private simulateApiCall(): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, 1500));
  }
}
// USAGE EXAMPLE - Component using Service Locator
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-service-locator-demo',
  template: `
    <div class="demo-container">
      <h2>Service Locator Pattern Demo</h2>
      
      <!-- Plugin Management -->
      <div class="section">
        <h3>Plugin Management</h3>
        <div class="plugins">
          <button (click)="loadAnalyticsPlugin()">Load Analytics Plugin</button>
          <button (click)="loadChatPlugin()">Load Chat Plugin</button>
          <button (click)="listPlugins()">List Plugins</button>
        </div>
        
        <div class="plugin-actions" *ngIf="activePlugins.length > 0">
          <h4>Active Plugins:</h4>
          <div *ngFor="let plugin of activePlugins" class="plugin-item">
            {{ plugin.name }} v{{ plugin.version }} ({{ plugin.state }})
            <button (click)="executeAnalytics()" *ngIf="plugin.name === 'analytics'">
              Track Event
            </button>
            <button (click)="executeChat()" *ngIf="plugin.name === 'chat'">
              Show Chat
            </button>
          </div>
        </div>
      </div>
      
      <!-- Payment Processing -->
      <div class="section">
        <h3>Payment Processing</h3>
        <div class="payment-options">
          <select [(ngModel)]="selectedProvider">
            <option value="">Select Payment Method</option>
            <option *ngFor="let provider of availableProviders" [value]="provider">
              {{ provider }}
            </option>
          </select>
          
          <input type="number" [(ngModel)]="paymentAmount" placeholder="Amount" />
          
          <select [(ngModel)]="paymentCurrency">
            <option value="USD">USD</option>
            <option value="EUR">EUR</option>
            <option value="GBP">GBP</option>
          </select>
          
          <button (click)="processPayment()" [disabled]="!selectedProvider || !paymentAmount">
            Process Payment
          </button>
        </div>
        
        <div class="payment-result" *ngIf="paymentResult">
          <h4>Payment Result:</h4>
          <p>Status: {{ paymentResult.success ? 'Success' : 'Failed' }}</p>
          <p *ngIf="paymentResult.transactionId">
            Transaction ID: {{ paymentResult.transactionId }}
          </p>
          <p *ngIf="paymentResult.error">Error: {{ paymentResult.error }}</p>
          <p>Fee: ${{ transactionFee.toFixed(2) }}</p>
        </div>
      </div>
      
      <!-- Service Locator Status -->
      <div class="section">
        <h3>Service Locator Status</h3>
        <p>Registered Services: {{ registeredServices.length }}</p>
        <ul>
          <li *ngFor="let service of registeredServices">{{ service }}</li>
        </ul>
      </div>
    </div>
  `,
  styles: [`
    .demo-container {
      padding: 20px;
      max-width: 800px;
      margin: 0 auto;
    }
    
    .section {
      margin-bottom: 30px;
      padding: 20px;
      border: 1px solid #ddd;
      border-radius: 8px;
    }
    
    h3 {
      margin-top: 0;
      color: #333;
    }
    
    button {
      margin: 5px;
      padding: 10px 20px;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    
    button:hover {
      background: #0056b3;
    }
    
    button:disabled {
      background: #ccc;
      cursor: not-allowed;
    }
    
    select, input {
      margin: 5px;
      padding: 8px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }
    
    .plugin-item {
      padding: 10px;
      margin: 5px 0;
      background: #f5f5f5;
      border-radius: 4px;
    }
    
    .payment-result {
      margin-top: 20px;
      padding: 15px;
      background: #f0f8ff;
      border-radius: 4px;
    }
  `]
})
export class ServiceLocatorDemoComponent implements OnInit {
  activePlugins: any[] = [];
  availableProviders: string[] = [];
  selectedProvider: string = '';
  paymentAmount: number = 100;
  paymentCurrency: string = 'USD';
  paymentResult: PaymentResult | null = null;
  transactionFee: number = 0;
  registeredServices: string[] = [];
  
  constructor(
    private serviceLocator: ServiceLocator,
    private pluginManager: PluginManager,
    private paymentManager: PaymentManager
  ) {}
  
  ngOnInit(): void {
    this.loadAvailableProviders();
    this.updateRegisteredServices();
  }
  
  // Plugin Management
  async loadAnalyticsPlugin(): Promise<void> {
    try {
      const plugin = new AnalyticsPlugin();
      await this.pluginManager.registerPlugin(plugin);
      this.updatePluginList();
      this.updateRegisteredServices();
      console.log('Analytics plugin loaded successfully');
    } catch (error) {
      console.error('Failed to load analytics plugin:', error);
    }
  }
  
  async loadChatPlugin(): Promise<void> {
    try {
      const plugin = new ChatPlugin();
      await this.pluginManager.registerPlugin(plugin);
      this.updatePluginList();
      this.updateRegisteredServices();
      console.log('Chat plugin loaded successfully');
    } catch (error) {
      console.error('Failed to load chat plugin:', error);
    }
  }
  
  listPlugins(): void {
    this.updatePluginList();
    console.log('Active plugins:', this.activePlugins);
  }
  
  async executeAnalytics(): Promise<void> {
    try {
      await this.pluginManager.executePlugin('analytics', {
        action: 'button_click',
        category: 'demo',
        label: 'service_locator_test',
        value: 1
      });
      console.log('Analytics event tracked');
    } catch (error) {
      console.error('Failed to execute analytics plugin:', error);
    }
  }
  
  async executeChat(): Promise<void> {
    try {
      await this.pluginManager.executePlugin('chat', {
        action: 'show'
      });
      console.log('Chat widget shown');
    } catch (error) {
      console.error('Failed to execute chat plugin:', error);
    }
  }
  
  // Payment Processing
  loadAvailableProviders(): void {
    this.availableProviders = this.paymentManager.getAvailableProviders();
  }
  
  async processPayment(): Promise<void> {
    if (!this.selectedProvider || !this.paymentAmount) {
      return;
    }
    
    // Calculate fee
    this.transactionFee = this.paymentManager.getProviderFee(
      this.selectedProvider,
      this.paymentAmount
    );
    
    // Prepare payment details based on provider
    const paymentDetails = this.getPaymentDetails(this.selectedProvider);
    
    // Process payment
    this.paymentResult = await this.paymentManager.processPayment(
      this.selectedProvider,
      this.paymentAmount,
      this.paymentCurrency,
      paymentDetails
    );
  }
  
  private getPaymentDetails(provider: string): any {
    switch (provider) {
      case 'paypal':
        return { email: 'user@example.com', password: 'secure123' };
      case 'stripe':
        return { cardNumber: '4242424242424242', cvv: '123', expiryDate: '12/25' };
      case 'creditcard':
        return {
          cardNumber: '1234567890123456',
          cardholderName: 'John Doe',
          cvv: '456',
          expiryMonth: '12',
          expiryYear: '2025'
        };
      default:
        return {};
    }
  }
  
  // Service Locator Status
  private updatePluginList(): void {
    this.activePlugins = this.pluginManager.listPlugins();
  }
  
  private updateRegisteredServices(): void {
    this.registeredServices = this.serviceLocator.listServices();
  }
}
```