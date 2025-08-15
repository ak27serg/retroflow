# RetroFlow

**Transform team reflection into actionable insights**

RetroFlow is a collaborative remote retrospective platform that eliminates the friction of remote retrospectives by providing a structured, real-time collaborative environment. The platform transforms scattered team feedback into prioritized, actionable insights through guided phases of reflection, grouping, and voting.

## 🚀 Features

### Complete Retrospective Workflow
- **Setup Phase**: Host controls and participant management
- **Input Phase**: Private response entry ("What went well" / "What didn't go well")
- **Grouping Phase**: Collaborative drag & drop card clustering with 70% overlap detection
- **Voting Phase**: Distribute 4 votes across groups and individual cards
- **Results Phase**: Vote-ordered display with full-screen presentation mode

### Real-time Collaboration
- **WebSocket connections** for live updates
- **Drag & drop grouping** with visual feedback
- **Real-time voting** with instant vote tallies
- **Presence awareness** showing online participants

### Professional Features
- **Full-screen presentation mode** with keyboard navigation
- **Export functionality** for results
- **Action items management**
- **Mobile-responsive design**
- **Professional UI/UX** with dark mode support

## 🏗️ Architecture

### Frontend
- **Next.js 15** with TypeScript
- **Tailwind CSS** for styling
- **@dnd-kit** for drag and drop
- **Socket.io client** for real-time features
- **Framer Motion** for animations

### Backend
- **Node.js** with Express
- **Socket.io** WebSocket server
- **TypeScript** for type safety
- **Prisma ORM** for database management
- **Redis** for session management

### Database
- **PostgreSQL** for persistent data
- **Redis** for real-time state and sessions
- **Docker containers** for easy deployment

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- Docker & Docker Compose
- npm or yarn

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/ak27serg/retroflow.git
   cd retroflow
   ```

2. **Start the backend services**
   ```bash
   cd retroflow-backend
   docker-compose up -d
   npm install
   npm run setup
   npm run dev
   ```

3. **Start the frontend**
   ```bash
   cd retroflow-frontend
   npm install
   npm run dev
   ```

4. **Access the application**
   - Frontend: http://localhost:3002
   - Backend API: http://localhost:3001
   - Database Studio: http://localhost:5555 (run `npx prisma studio`)

## 📖 Usage

### For Hosts
1. Visit the application URL
2. Click "Create Session"
3. Enter session details and your information
4. Share the invite code with participants
5. Guide the team through the 5-phase workflow

### For Participants
1. Visit the application URL
2. Click "Join Session"
3. Enter the invite code provided by the host
4. Complete your profile and join the retrospective

## 🛠️ Development

### Project Structure
```
retroflow/
├── retroflow-frontend/     # Next.js frontend application
├── retroflow-backend/      # Node.js backend API
├── DEMO_READY.md          # Demo instructions and features
├── retrospectives-product-spec.md  # Product specification
└── README.md              # This file
```

### Environment Setup
1. Copy `.env.example` files in both frontend and backend directories
2. Update database and Redis connection strings
3. Configure CORS origins for your deployment environment

### Database Management
```bash
# View database with Prisma Studio
cd retroflow-backend && npx prisma studio

# Reset database
npx prisma db push --force-reset

# Generate Prisma client
npx prisma generate
```

## 🚢 Deployment

### Hosting Options
- **Railway**: Easy deployment with built-in PostgreSQL and Redis
- **Render**: Git-based deployment with managed databases
- **Fly.io**: Global edge deployment
- **AWS/GCP/Azure**: Enterprise-scale deployment
- **Self-hosted**: Docker Compose on VPS

### Docker Deployment
```bash
# Backend services (PostgreSQL + Redis)
cd retroflow-backend && docker-compose up -d

# Frontend and backend can be containerized for production
```

## 🎯 Demo

The application includes a complete demo setup with:
- ✅ Full workflow testing
- ✅ Real-time collaboration features
- ✅ Database with sample data
- ✅ Professional presentation mode

See `DEMO_READY.md` for detailed demo instructions.

## 📊 Technical Specifications

### Database Schema
- **Sessions**: Retrospective session management
- **Participants**: User management and roles
- **Responses**: Feedback entries with categories
- **Groups**: Clustered feedback with positioning
- **Votes**: Priority voting system

### Real-time Features
- WebSocket event architecture
- Optimistic UI updates
- Conflict resolution strategies
- Presence awareness

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Links

- **Repository**: https://github.com/ak27serg/retroflow
- **Issues**: https://github.com/ak27serg/retroflow/issues
- **Documentation**: See `/docs` directory for detailed documentation

---

**RetroFlow** - Making remote retrospectives engaging and actionable for distributed teams.