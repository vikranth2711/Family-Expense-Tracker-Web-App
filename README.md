# 👨‍👩‍👧‍👦 Family Expense Tracker

A full-stack web application for tracking and managing family expenses with group splitting, built with React, Node.js, MongoDB, and deployed with complete DevOps automation.

![MERN Stack](https://img.shields.io/badge/MERN-Stack-blue)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![AWS](https://img.shields.io/badge/AWS-Free%20Tier-orange)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple)
![Status](https://img.shields.io/badge/Status-Live-brightgreen)

## 🌐 Live Demo

**Frontend**: http://3.226.196.225  
**Backend API**: http://3.226.196.225:5000  
**Grafana Monitoring**: http://3.226.196.225:3001 (admin/admin)  
**Prometheus Metrics**: http://3.226.196.225:9090

---

## ✨ Features

### Core Functionality
- 🔐 **User Authentication** - JWT-based secure login/registration
- 👥 **Group Management** - Create and manage family/friend groups
- 💰 **Expense Tracking** - Add, edit, delete expenses with categories
- 📊 **Smart Splitting** - Automatic expense splitting among group members
- 💳 **Balance Calculations** - Real-time balance tracking and settlements
- 📈 **Reports & Analytics** - Visual charts and expense insights
- 🎨 **Dark Mode** - Beautiful UI with light/dark theme toggle
- 📱 **Responsive Design** - Works seamlessly on all devices

### DevOps Features
- 🐳 **Docker Containerization** - Multi-container setup with Docker Compose
- ☁️ **AWS Deployment** - Terraform-managed infrastructure on AWS EC2
- 📊 **Monitoring Stack** - Prometheus + Grafana for metrics and visualization
- 🔄 **CI/CD Ready** - Jenkins pipeline configuration included
- 🚀 **Zero-downtime Deployment** - Container orchestration with health checks

---

## 🏗️ Architecture

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐
│                 │      │                 │      │                 │
│  React Frontend │─────▶│  Node.js API    │─────▶│  MongoDB Atlas  │
│  (Port 80)      │      │  (Port 5000)    │      │  (Cloud)        │
│                 │      │                 │      │                 │
└─────────────────┘      └─────────────────┘      └─────────────────┘
         │                        │
         │                        │
         ▼                        ▼
┌─────────────────┐      ┌─────────────────┐
│                 │      │                 │
│   Grafana       │◀─────│   Prometheus    │
│   (Port 3001)   │      │   (Port 9090)   │
│                 │      │                 │
└─────────────────┘      └─────────────────┘
```

---

## 🛠️ Technology Stack

### Frontend
- **React 18** - Modern UI library with hooks
- **Tailwind CSS** - Utility-first CSS framework
- **Chart.js** - Data visualization
- **Axios** - HTTP client
- **React Router** - Client-side routing
- **Framer Motion** - Smooth animations

### Backend
- **Node.js** - JavaScript runtime
- **Express.js** - Web application framework
- **MongoDB** - NoSQL database
- **Mongoose** - MongoDB ODM
- **JWT** - Authentication tokens
- **bcrypt** - Password hashing

### DevOps & Infrastructure
- **Docker** - Containerization platform
- **Docker Compose** - Multi-container orchestration
- **Terraform** - Infrastructure as Code (IaC)
- **AWS EC2** - Cloud compute instances
- **Prometheus** - Metrics collection
- **Grafana** - Monitoring dashboards
- **Jenkins** - CI/CD automation
- **Ansible** - Configuration management (optional)

---

## 🚀 Quick Start (Local Development)

### Prerequisites
- Node.js 18+ installed
- Docker & Docker Compose installed
- MongoDB Atlas account (free tier) or local MongoDB

### 1. Clone the Repository
```bash
git clone https://github.com/vikranth2711/Family-Expense-Tracker-Web-App.git
cd Family-Expense-Tracker-Web-App
```

### 2. Start with Docker Compose
```bash
# Start all services (backend, frontend, monitoring)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down
```

### 3. Access the Application
- **Frontend**: http://localhost:3000
- **Backend**: http://localhost:5000
- **Grafana**: http://localhost:3001 (admin/admin)
- **Prometheus**: http://localhost:9090

---

## 🌩️ Production Deployment (AWS)

### Prerequisites
- AWS Account with Free Tier
- AWS CLI configured
- Terraform installed
- SSH key pair generated

### 1. Configure Terraform Variables
```bash
cd terraform
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your values:
# - mongodb_uri (MongoDB Atlas connection string)
# - jwt_secret (generate with: openssl rand -base64 32)
# - ssh_public_key_path (path to your id_rsa.pub)
```

### 2. Deploy Infrastructure
```bash
cd terraform
terraform init
terraform plan
terraform apply -auto-approve
```

This creates:
- VPC with public subnet
- EC2 t3.micro instance (Free Tier eligible)
- Security groups (ports: 22, 80, 443, 5000, 3001, 9090)
- Elastic IP address

### 3. Deploy Application
The application is automatically deployed using Docker Compose on the EC2 instance.

### 4. Access Production
- Frontend: `http://<your-elastic-ip>`
- Backend: `http://<your-elastic-ip>:5000`
- Grafana: `http://<your-elastic-ip>:3001`

### 5. Destroy Infrastructure (Save AWS Credits!)
```bash
cd terraform
terraform destroy -auto-approve
```

---

## 📦 Docker Images

Pre-built images available on Docker Hub:
- **Backend**: `vikranth2711/expense-tracker-backend:latest`
- **Frontend**: `vikranth2711/expense-tracker-frontend:latest`

Build your own:
```bash
# Backend
docker build -t your-username/expense-tracker-backend:latest ./backend

# Frontend
docker build -t your-username/expense-tracker-frontend:latest ./frontend

# Push to Docker Hub
docker push your-username/expense-tracker-backend:latest
docker push your-username/expense-tracker-frontend:latest
```

---

## 🔧 Configuration

### Environment Variables

#### Backend (server.js)
```env
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/dbname
JWT_SECRET=your-secret-key-here
PORT=5000
NODE_ENV=production
```

#### Frontend (Build-time)
```env
REACT_APP_API_URL=http://your-backend-url:5000
```

---

## 📊 Monitoring & Metrics

### Prometheus
- Scrapes metrics from Node Exporter
- Default scrape interval: 15s
- Access: http://localhost:9090 or http://<elastic-ip>:9090

### Grafana
- Pre-configured dashboards for system metrics
- Default credentials: admin/admin (change on first login)
- Data source: Prometheus (auto-configured)
- Access: http://localhost:3001 or http://<elastic-ip>:3001

### Available Metrics
- CPU usage
- Memory consumption
- Disk I/O
- Network traffic
- Container health status

---

## 🔄 CI/CD Pipeline (Jenkins)

### Setup Jenkins (Optional)
```bash
# Jenkins is configured in Jenkinsfile
# To use:
1. Install Jenkins on server
2. Create new pipeline job
3. Point to Jenkinsfile in repository
4. Configure GitHub webhook
```

### Pipeline Stages
1. **Checkout** - Clone repository
2. **Build** - Build Docker images
3. **Test** - Run test suites
4. **Push** - Push images to Docker Hub
5. **Deploy** - Deploy to EC2 instance
6. **Health Check** - Verify deployment

---

## 📁 Project Structure

```
Family-Expense-Tracker-Web-App/
├── backend/                   # Node.js Express API
│   ├── controllers/           # Business logic
│   ├── models/                # MongoDB schemas
│   ├── routes/                # API endpoints
│   ├── middleware/            # Auth middleware
│   ├── scripts/               # DB initialization
│   ├── Dockerfile             # Backend container
│   └── server.js              # Entry point
├── frontend/                  # React application
│   ├── public/                # Static assets
│   ├── src/
│   │   ├── components/        # React components
│   │   ├── pages/             # Page components
│   │   ├── context/           # Context providers
│   │   ├── api/               # API client
│   │   └── App.js             # Main app
│   ├── Dockerfile             # Frontend container
│   └── nginx.conf             # Nginx configuration
├── terraform/                 # Infrastructure as Code
│   ├── main.tf                # AWS resources
│   ├── variables.tf           # Input variables
│   ├── outputs.tf             # Output values
│   └── terraform.tfvars       # Variable values (gitignored)
├── ansible/                   # Configuration management
│   ├── playbook.yml           # Ansible playbook
│   └── inventory.ini          # Host inventory
├── monitoring/                # Monitoring configs
│   ├── prometheus/
│   │   └── prometheus.yml     # Prometheus config
│   └── grafana/
│       └── dashboards/        # Grafana dashboards
├── jenkins/                   # CI/CD configuration
│   └── scripts/               # Deployment scripts
├── docker-compose.yml         # Local development
├── docker-compose.prod.yml    # Production deployment
├── Jenkinsfile                # Jenkins pipeline
└── README.md                  # This file
```

---

## 🧪 Testing

### Backend Tests
```bash
cd backend
npm install
npm test
```

### Frontend Tests
```bash
cd frontend
npm install
npm test
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📝 API Documentation

### Authentication Endpoints
```
POST /api/auth/register  - Register new user
POST /api/auth/login     - Login user
GET  /api/auth/me        - Get current user
```

### Group Endpoints
```
POST   /api/groups              - Create group
GET    /api/groups/mine         - List user's groups
GET    /api/groups/:id          - Get group details
POST   /api/groups/join         - Join group
POST   /api/groups/:id/members  - Add member
DELETE /api/groups/:id/members  - Remove member
```

### Expense Endpoints
```
GET    /api/expenses?groupId=<id>  - Get group expenses
POST   /api/expenses               - Create expense
GET    /api/expenses/:id           - Get expense details
PUT    /api/expenses/:id           - Update expense
DELETE /api/expenses/:id           - Delete expense
GET    /api/expenses/balances      - Get balance calculations
```

### Category Endpoints
```
GET    /api/categories?groupId=<id>  - Get categories
POST   /api/categories                - Create category
PUT    /api/categories/:id            - Update category
DELETE /api/categories/:id            - Delete category
```

---

## 🐛 Troubleshooting

### Common Issues

**Backend won't connect to MongoDB**
```bash
# Check MongoDB Atlas connection string
# Ensure IP whitelist includes 0.0.0.0/0 for testing
# Verify credentials in .env file
```

**Frontend can't reach backend**
```bash
# Check REACT_APP_API_URL in environment
# Verify backend is running on port 5000
# Check CORS settings in backend
```

**Docker container crashes**
```bash
# View logs
docker logs <container-name>

# Check container status
docker ps -a

# Restart container
docker-compose restart <service-name>
```

**AWS deployment fails**
```bash
# Verify AWS credentials
aws configure list

# Check Terraform state
cd terraform && terraform show

# View EC2 instance logs
ssh -i <key>.pem ubuntu@<ip> "sudo docker logs <container>"
```

---

## 💰 AWS Cost Optimization

### Free Tier Eligible Resources
- **EC2**: 750 hours/month of t3.micro (12 months)
- **EBS**: 30GB of storage
- **Data Transfer**: 15GB outbound per month
- **MongoDB Atlas**: 512MB storage (forever free)

### Cost-Saving Tips
1. **Stop instances when not in use**
   ```bash
   aws ec2 stop-instances --instance-ids <instance-id>
   ```

2. **Use Elastic IP only when instance is running** (charges when not attached)

3. **Monitor usage in AWS Cost Explorer**

4. **Set up billing alerts** at $5, $10 thresholds

5. **Destroy infrastructure after testing**
   ```bash
   terraform destroy -auto-approve
   ```

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

## 👤 Author

**Vikranth**
- GitHub: [@vikranth2711](https://github.com/vikranth2711)
- Repository: [Family-Expense-Tracker-Web-App](https://github.com/vikranth2711/Family-Expense-Tracker-Web-App)

---

## 🙏 Acknowledgments

- MongoDB Atlas for free database hosting
- AWS Free Tier for cloud infrastructure
- Docker Hub for free image hosting
- Prometheus & Grafana communities

---

## 📞 Support

For issues and questions:
- Create an issue in the GitHub repository
- Check existing issues for solutions
- Review troubleshooting section above

---

**Built with ❤️ using React, Node.js, and DevOps best practices**
