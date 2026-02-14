# HR Management Microservices System

A distributed HR management system built with microservices architecture using Spring Boot 3.2+, Angular 17, and PostgreSQL.

## 🏗️ Architecture Overview

### Backend Services
- **Eureka Server** (Port: 8761) - Service discovery and registration
- **API Gateway** (Port: 8080) - Single entry point, JWT validation, routing, CORS
- **Auth Service** (Port: 8081) - User authentication, JWT token generation, role management
- **Employee Service** (Port: 8082) - Employee CRUD operations, search, and filtering
- **Leave Service** (Port: 8083) - Leave request management with approval workflow
- **Notification Service** (Port: 8084) - Email notifications via RabbitMQ

### Frontend
- **Angular Frontend** (Port: 4200) - Standalone components, PrimeNG UI, role-based dashboards

### Infrastructure
- **PostgreSQL 16** - One database per service
- **RabbitMQ** - Async messaging for notifications
- **Docker & Docker Compose** - Containerization and orchestration

## 🎯 Key Features

### Authentication & Security
- JWT-based authentication with refresh tokens
- Role-based access control (EMPLOYEE, HR_MANAGER, ADMIN)
- BCrypt password encryption
- CORS configuration

### Employee Management
- Full CRUD operations with pagination
- Search and filtering by name, department, position
- Manager-employee relationships

### Leave Management
- Multiple leave types (PAID_LEAVE, SICK_LEAVE, UNPAID_LEAVE, MATERNITY_LEAVE)
- Annual quota tracking (25 paid days, 10 sick days default)
- Multi-level approval workflow (Employee → Manager → HR)
- Balance calculation with overlap prevention

### Notifications
- Asynchronous email notifications
- Customizable email templates
- Failed notification logging

## 📋 Prerequisites

- Docker & Docker Compose
- Java 21 (for local development)
- Node.js 18+ & Angular CLI 17+ (for frontend development)
- At least 4GB RAM for Docker containers

## 🚀 Quick Start

### Method 1: Docker Compose (Recommended)

```bash
# Clone and navigate to the project
git clone <your-repo-url>
cd hr-microservices

# Start all services
docker-compose up -d

# Check services health
docker-compose ps

# View logs
docker-compose logs api-gateway
```

### Method 2: Local Development

```bash
# 1. Start infrastructure (PostgreSQL, RabbitMQ)
docker-compose -f docker-compose.yml up postgresql rabbitmq eureka-server -d

# 2. Start services in order:
cd auth-service && mvn spring-boot:run
cd ../employee-service && mvn spring-boot:run
cd ../leave-service && mvn spring-boot:run
cd ../notification-service && mvn spring-boot:run
cd ../api-gateway && mvn spring-boot:run

# 3. Start frontend
cd frontend-angular
npm install
npm start
```

## 🌐 Access the Application

| Service | URL | Description |
|---------|-----|-------------|
| Front End | http://localhost:4200 | Angular Application |
| API Gateway | http://localhost:8080 | Main API Endpoint |
| Eureka Dashboard | http://localhost:8761 | Service Registry |
| RabbitMQ Management | http://localhost:15672 | Queue Management (guest/guest) |
| PostgreSQL AuthDB | localhost:5432 | Database (authservice/authpass) |

### Default Credentials

| Role | Username | Email | Password |
|------|----------|-------|----------|
| Admin | admin | admin@hrms.com | admin123 |
| HR Manager | hr_manager | hr@hrms.com | hr123 |
| Employee | employee | emp@example.com | emp123 |

## 📚 API Documentation

### Authentication Endpoints (`/api/auth`)

```bash
# User Registration
POST /api/auth/register
Content-Type: application/json
{
  "username": "john_doe",
  "email": "john.doe@company.com",
  "password": "password123",
  "firstName": "John",
  "lastName": "Doe",
  "role": "EMPLOYEE"
}

# User Login
POST /api/auth/login
Content-Type: application/json
{
  "username": "admin",
  "password": "admin123"
}

# Token Validation
GET /api/auth/validate
Authorization: Bearer <jwt_token>
```

### Employee Endpoints (`/api/employees`)

```bash
# List Employees (with pagination)
GET /api/employees?page=0&size=10&search=John&department=IT

# Get Employee by ID
GET /api/employees/{id}

# Create Employee
POST /api/employees
Content-Type: application/json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@company.com",
  "phone": "+1234567890",
  "address": "123 Main St",
  "dateOfBirth": "1990-01-01",
  "hireDate": "2023-01-15",
  "position": "Software Developer",
  "department": "IT",
  "salary": 75000.00,
  "managerId": null
}

# Update Employee
PUT /api/employees/{id}

# Delete Employee
DELETE /api/employees/{id}
```

### Leave Endpoints (`/api/leaves`)

```bash
# Get Employee's Leaves
GET /api/leaves/employee/{employeeId}

# Submit Leave Request
POST /api/leaves
Authorization: Bearer <jwt_token>
Content-Type: application/json
{
  "employeeId": 1,
  "startDate": "2024-02-01",
  "endDate": "2024-02-03",
  "leaveType": "PAID_LEAVE",
  "reason": "Vacation"
}

# Approve/Reject Leave
PUT /api/leaves/{id}/approve
PUT /api/leaves/{id}/reject

# Get Leave Balance
GET /api/leaves/{employeeId}/balance
```

## 🗄️ Database Schema

### Auth Service Database
- `users` - User accounts with roles
- `refresh_tokens` - JWT refresh token storage

### Employee Service Database
- `employees` - Employee information
- `departments` - Department list

### Leave Service Database
- `leaves` - Leave requests
- `leave_balances` - Annual leave quotas

### Notification Service Database
- `notifications` - Notification history

## 🐳 Docker Commands

```bash
# Start all services
docker-compose up -d

# Start specific service
docker-compose up -d auth-service

# View logs
docker-compose logs -f auth-service

# Rebuild and restart service
docker-compose up -d --build auth-service

# Stop all services
docker-compose down

# Clean up volumes
docker-compose down -v
```

## 🧪 Testing

### Unit Tests
```bash
# Run tests for specific service
cd auth-service
mvn test

cd employee-service
mvn test
```

### Integration Tests
```bash
# Test with local infrastructure
docker-compose -f docker-compose.test.yml up --abort-on-container-exit
```

## 🔧 Development

### Adding New Microservice

1. Create service directory: `mkdir new-service`
2. Add service structure with standard packages
3. Configure application.yml with service-specific properties
4. Add to docker-compose.yml
5. Register with Eureka Server

### Frontend Development

```bash
cd frontend-angular

# Install dependencies
npm install

# Development server
npm start

# Build for production
npm run build --prod

# Run tests
npm test
```

## 🛠️ Configuration

### Environment Variables

Create `.env` file in root directory:

```env
# Database Configuration
POSTGRESQL_AUTH_PASSWORD=authpass
POSTGRESQL_EMPLOYEE_PASSWORD=emppass
POSTGRESQL_LEAVE_PASSWORD=leavepass
POSTGRESQL_NOTIFICATION_PASSWORD=notifypass

# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key-minimum-64-characters-long
JWT_EXPIRATION=86400000

# Email Configuration (for local testing)
SPRING_MAIL_HOST=smtp.gmail.com
SPRING_MAIL_PORT=587
SPRING_MAIL_USERNAME=your-email@gmail.com
SPRING_MAIL_PASSWORD=your-app-password

# RabbitMQ
RABBITMQ_DEFAULT_USER=guest
RABBITMQ_DEFAULT_PASS=guest
```

### Spring Profiles

Services use `application.yml` configuration with environment variables for:
- Database connections
- Eureka client settings
- RabbitMQ configuration
- JWT settings

## 🔍 Monitoring & Health Checks

### Actuator Endpoints
```bash
# Health Check
GET /actuator/health

# Service Info
GET /actuator/info

# Metrics
GET /actuator/metrics
```

### Eureka Dashboard
- View all registered services: http://localhost:8761

## 🐛 Troubleshooting

### Common Issues

1. **Port conflicts**
   ```bash
   # Check if ports are in use
   netstat -tulpn | grep :8080
   # Stop conflicting services or change ports in docker-compose.yml
   ```

2. **Database connection issues**
   ```bash
   # Check database logs
   docker-compose logs postgresql
   # Verify connection string in application.yml
   ```

3. **Service not registering with Eureka**
   ```bash
   # Check service logs
   docker-compose logs auth-service
   # Verify Eureka client configuration
   ```

4. **Frontend build issues**
   ```bash
   cd frontend-angular
   # Clear node_modules
   rm -rf node_modules package-lock.json
   npm cache clean --force
   npm install
   ```

### Logs and Debugging

```bash
# View all service logs
docker-compose logs -f

# View specific service logs
docker-compose logs -f auth-service

# Debug mode (services)
cd auth-service
mvn spring-boot:run -Dspring-boot.run.profiles=debug
```

### Reset System

```bash
# Stop and remove all containers
docker-compose down -v

# Remove dangling images
docker image prune -f

# Restart fresh
docker-compose up -d --build
```

## 🎯 Architecture Decisions

### Why Microservices?
- Scalability: Scale individual services based on load
- Technology diversity: Use different technologies per service
- Fault isolation: Failure in one service doesn't affect others
- Team autonomy: Independent deployment and development

### Why Spring Boot 3.2+?
- Latest Spring features and performance improvements
- Better reactive programming support
- Enhanced security with OAuth2/JWT
- Improved container compatibility

### Why Angular 17?
- Standalone components for better tree shaking
- Improved performance with Ivy compiler
- Better TypeScript support
- Standalone-first approach

### Why PostgreSQL?
- ACID compliance for transactional data
- Advanced indexing and querying
- Excellent JSON support for complex data
- Strong community support

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Submit a pull request

### Code Standards
- Use Spring Boot conventions
- Follow REST API guidelines
- Implement proper error handling
- Add comprehensive documentation
- Use DTOs for API contracts

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For support, please contact the development team or create an issue in the repository.

---

**Note**: This is a prototype system. For production deployment, additional security measures, monitoring, and scaling configurations should be implemented.