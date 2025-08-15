# RetroFlow: Collaborative Remote Retrospective Platform

## Product Overview

**Project Name:** RetroFlow  
**Tagline:** "Transform team reflection into actionable insights"

**Core Value Proposition:**  
RetroFlow eliminates the friction of remote retrospectives by providing a structured, real-time collaborative environment that maintains participant privacy during input while fostering open discussion during analysis. The platform transforms scattered team feedback into prioritized, actionable insights through guided phases of reflection, grouping, and voting.

## Product Specification

### 1. Core User Journey

#### Phase 1: Session Setup & Join
- Host creates a new retrospective session
- System generates unique invite link with session ID
- Participants join via link, enter display name, select avatar
- Host can see participant list and start session when ready

#### Phase 2: Private Input Phase
- Users see "What went well" and "What didn't go well" input areas
- Clicking opens text editor for private response entry
- While typing, host sees blurred preview indicating activity
- Users can add multiple responses per category
- Users can edit/delete their own responses before submission
- Timer countdown or manual progression by host

#### Phase 3: Reveal & Grouping Phase
- All responses become visible as draggable cards
- Participants collaboratively drag related cards into clusters
- Real-time updates show all drag operations
- Groups can be labeled by any participant
- Visual indication of who's currently dragging items

#### Phase 4: Voting Phase
- Each user receives 4 votes to distribute
- Votes can be allocated to any grouped cluster (not individual cards)
- Real-time vote tallies visible to all
- Users can redistribute votes until phase ends

#### Phase 5: Results & Action Items
- Groups displayed in descending order by vote count
- Export functionality for results
- Optional action item creation from top-voted groups

### 2. Technical Architecture

#### Frontend Stack
- **Framework:** Next.js 14 (React 18) with TypeScript
- **Styling:** Tailwind CSS with Headless UI components
- **State Management:** Zustand for client state, React Query for server state
- **Real-time:** Socket.io client
- **Drag & Drop:** @dnd-kit/core for accessible drag and drop
- **Animations:** Framer Motion for smooth transitions
- **Build Tool:** Vite for fast development

#### Backend Stack
- **Runtime:** Node.js with Express.js
- **Language:** TypeScript
- **Database:** PostgreSQL with Prisma ORM
- **Real-time:** Socket.io server
- **Authentication:** Session-based with Redis store
- **Validation:** Zod for schema validation
- **Rate Limiting:** Express Rate Limit
- **Deployment:** Docker containers

#### Infrastructure
- **Hosting:** Railway or Render for MVP, AWS/Vercel for scale
- **Database:** Managed PostgreSQL (Supabase/Railway)
- **Redis:** Managed Redis for sessions and real-time state
- **CDN:** Cloudflare for static assets
- **Monitoring:** Basic logging with structured JSON

### 3. Database Schema

```sql
-- Sessions table
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_code VARCHAR(8) UNIQUE NOT NULL,
    host_id UUID NOT NULL,
    title VARCHAR(255) DEFAULT 'Retrospective',
    current_phase VARCHAR(20) DEFAULT 'setup',
    timer_duration INTEGER DEFAULT 600, -- seconds
    timer_end_time TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    settings JSONB DEFAULT '{}'
);

-- Participants table
CREATE TABLE participants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
    display_name VARCHAR(50) NOT NULL,
    avatar_id VARCHAR(20) NOT NULL,
    is_host BOOLEAN DEFAULT FALSE,
    joined_at TIMESTAMP DEFAULT NOW(),
    last_active TIMESTAMP DEFAULT NOW(),
    socket_id VARCHAR(50)
);

-- Responses table
CREATE TABLE responses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
    participant_id UUID REFERENCES participants(id) ON DELETE CASCADE,
    category VARCHAR(20) NOT NULL, -- 'went_well' or 'didnt_go_well'
    content TEXT NOT NULL,
    group_id UUID,
    position_x INTEGER DEFAULT 0,
    position_y INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Groups table
CREATE TABLE groups (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
    label VARCHAR(100),
    color VARCHAR(7) DEFAULT '#3B82F6',
    position_x INTEGER DEFAULT 0,
    position_y INTEGER DEFAULT 0,
    vote_count INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Votes table
CREATE TABLE votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES sessions(id) ON DELETE CASCADE,
    participant_id UUID REFERENCES participants(id) ON DELETE CASCADE,
    group_id UUID REFERENCES groups(id) ON DELETE CASCADE,
    vote_count INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(participant_id, group_id)
);

-- Indexes for performance
CREATE INDEX idx_sessions_invite_code ON sessions(invite_code);
CREATE INDEX idx_participants_session_id ON participants(session_id);
CREATE INDEX idx_responses_session_participant ON responses(session_id, participant_id);
CREATE INDEX idx_votes_session_participant ON votes(session_id, participant_id);
```

### 4. Real-time Collaboration Approach

#### Socket.io Event Architecture

**Connection Management:**
```typescript
// Client joins session
socket.emit('join_session', { sessionId, participantId })

// Server tracks active participants
socket.join(`session:${sessionId}`)
io.to(`session:${sessionId}`).emit('participant_joined', participantData)
```

**Phase Management:**
```typescript
// Host controls phase transitions
socket.emit('change_phase', { sessionId, phase: 'input' })
io.to(`session:${sessionId}`).emit('phase_changed', { phase, timerEnd })
```

**Real-time Features:**
- **Typing Indicators:** Broadcast blurred activity during input phase
- **Live Dragging:** Stream drag operations with position updates
- **Vote Updates:** Instant vote count synchronization
- **Presence Awareness:** Show online/offline participant status

#### State Synchronization Strategy

**Optimistic Updates:** Client immediately reflects user actions
**Conflict Resolution:** Last-write-wins for positioning, server authority for votes
**Reconciliation:** Periodic state sync to handle disconnections

### 5. UI/UX Flow by Phase

#### Setup Phase
```
┌─────────────────────────────────────┐
│  RetroFlow                         │
│                                     │
│  Session: team-retro-jan-2024      │
│  Participants (3/8):               │
│  ● Alex 👨‍💻  ● Sarah 👩‍🎨  ● Mike 🧔    │
│                                     │
│  Waiting for more participants...   │
│  Share: retro.flow/abc123          │
│                                     │
│  [Start Retrospective] (Host only)  │
└─────────────────────────────────────┘
```

#### Input Phase
```
┌─────────────────────────────────────┐
│  Input Phase - 8:42 remaining ⏱️   │
│                                     │
│  What went well? 😊                │
│  ┌─────────────────────────────────┐ │
│  │ [Click to add response...]      │ │
│  └─────────────────────────────────┘ │
│                                     │
│  What didn't go well? 😕            │
│  ┌─────────────────────────────────┐ │
│  │ [Click to add response...]      │ │
│  └─────────────────────────────────┘ │
│                                     │
│  Activity: ●○○ (3 typing)          │
└─────────────────────────────────────┘
```

#### Grouping Phase
```
┌─────────────────────────────────────┐
│  Grouping Phase - Drag to cluster   │
│                                     │
│  [Communication] [Process] [Tools]   │
│     ┌──────┐    ┌──────┐   ┌──────┐ │
│     │Stand-│    │Retro │   │Slack │ │
│     │ups   │    │cadence│   │usage │ │
│     └──────┘    └──────┘   └──────┘ │
│                                     │
│  Ungrouped:                         │
│  ┌──────┐ ┌──────┐ ┌──────┐         │
│  │Deploy│ │Bug   │ │Team  │         │
│  │issues│ │fixes │ │spirit│         │
│  └──────┘ └──────┘ └──────┘         │
└─────────────────────────────────────┘
```

#### Voting Phase
```
┌─────────────────────────────────────┐
│  Voting Phase - 4 votes remaining   │
│                                     │
│  Communication (2 votes) 🔵🔵       │
│  [Daily standups improved]          │
│  [Better async updates]             │
│                                     │
│  Process Issues (1 vote) 🔴         │
│  [Retrospective cadence unclear]    │
│  [Deploy process confusing]         │
│                                     │
│  Tools (1 vote) 🟡                 │
│  [Slack integration helpful]        │
│  [Need better monitoring]           │
└─────────────────────────────────────┘
```

#### Results Phase
```
┌─────────────────────────────────────┐
│  Results - Top Priorities           │
│                                     │
│  🥇 Communication (6 votes)         │
│     • Daily standups improved       │
│     • Better async updates          │
│                                     │
│  🥈 Process Issues (4 votes)        │
│     • Deploy process confusing      │
│     • Retrospective cadence unclear │
│                                     │
│  🥉 Tools (2 votes)                │
│     • Need better monitoring        │
│     • Slack integration helpful     │
│                                     │
│  [Export Results] [New Retro]      │
└─────────────────────────────────────┘
```

### 6. Security & Privacy Considerations

#### Data Privacy
- **Input Privacy:** Responses encrypted in transit, only revealed after input phase
- **Session Isolation:** Strict session boundaries, no cross-contamination
- **Data Retention:** Sessions auto-delete after 30 days, manual export available
- **Anonymous Mode:** Optional anonymous participation (no names stored)

#### Security Measures
- **Rate Limiting:** Prevent spam and abuse per IP/session
- **Input Validation:** Sanitize all user inputs, prevent XSS
- **Session Management:** Secure session tokens, automatic cleanup
- **CORS Policy:** Restrict origins, secure WebSocket connections

#### Session Management
- **Invite Links:** Time-limited, single-use options available
- **Host Controls:** Only host can advance phases, manage participants
- **Graceful Disconnection:** Reconnection handling, state preservation
- **Concurrent Limits:** Max participants per session (default: 15)

### 7. Optional Enhancements & Optimizations

#### MVP+ Features
- **Custom Templates:** Beyond "went well/didn't go well"
  - Start/Stop/Continue
  - Glad/Sad/Mad
  - Custom category creation
- **Action Item Tracking:** Convert insights to trackable tasks
- **Session History:** Save past retrospectives for trends
- **Team Analytics:** Aggregate insights across sessions

#### Advanced Features
- **AI Insights:** Automated grouping suggestions, sentiment analysis
- **Integration APIs:** Slack/Discord bots, calendar scheduling
- **Advanced Voting:** Weighted voting, dot voting variations
- **Multi-language:** I18n support for global teams
- **White-label:** Custom branding for enterprise clients

#### Performance Optimizations
- **Lazy Loading:** Paginated participant lists, virtual scrolling
- **Caching Strategy:** Redis caching for session state
- **CDN Assets:** Optimized avatar delivery
- **Database Indexing:** Query optimization for large sessions
- **WebSocket Scaling:** Redis adapter for horizontal scaling

#### Accessibility Enhancements
- **Screen Reader:** Full ARIA support, keyboard navigation
- **High Contrast:** Multiple theme options
- **Font Scaling:** Responsive text sizing
- **Color Blind:** Color-blind friendly indicators
- **Mobile First:** Touch-optimized drag and drop

### 8. MVP Development Timeline

**Week 1-2:** Core architecture, database setup, basic auth
**Week 3-4:** Real-time infrastructure, session management  
**Week 5-6:** Input phase UI/UX, response management
**Week 7-8:** Grouping phase, drag and drop functionality
**Week 9-10:** Voting system, results visualization
**Week 11-12:** Polish, testing, deployment pipeline

### 9. Success Metrics

**User Engagement:**
- Session completion rate (target: >80%)
- Average responses per participant (target: >3)
- Return usage rate (target: >40%)

**Technical Performance:**
- Page load time <2s
- Real-time latency <100ms
- 99.9% uptime

**Business Validation:**
- User satisfaction score >4.2/5
- Organic sharing rate >25%
- Session creation growth week-over-week

---

This specification provides a solid foundation for MVP development while outlining a clear path for future enhancements. The architecture balances simplicity with scalability, ensuring rapid iteration while maintaining room for growth.