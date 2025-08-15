# RetroFlow Complete Workflow Test

## Test URLs:
- Frontend: http://localhost:3002
- Backend API: http://localhost:3001

## Manual Testing Steps:

### 1. Create Session
1. Go to http://localhost:3002
2. Fill in "Start New Retro" form:
   - Session Title: "Test Sprint Retro"
   - Your Name: "Test Host"
   - Choose any avatar
3. Click "Create Session"
4. Should redirect to session page

### 2. Setup Phase
1. Should see setup phase with session details
2. Click "Start Input Phase" (host only)

### 3. Input Phase
1. Add responses in both columns:
   - "What went well": Add 2-3 positive responses
   - "What didn't go well": Add 2-3 negative responses
2. Test edit/delete functionality
3. Click "Move to Grouping" (host only)

### 4. Grouping Phase  
1. Drag cards close together (70% overlap) to form groups
2. Name the groups by clicking on group labels
3. Test ungrouping with double-click
4. Click "Move to Voting" (host only)

### 5. Voting Phase
1. Should see all groups AND individual cards as votable items
2. Distribute 4 votes across groups/cards
3. Test +/- buttons work
4. Click "View Results" (host only)

### 6. Results Phase
1. Should see results sorted by vote count
2. Test "Start Presentation" for full-screen view
3. Test navigation with arrow keys
4. Test "Export Results" 
5. Test "New Retro" button

## Expected Outcomes:
- ✅ Host automatically added as participant
- ✅ Real-time updates work
- ✅ Cards group properly with 70% overlap
- ✅ Individual cards are votable
- ✅ Results show with correct vote counts
- ✅ Presentation mode works
- ✅ No critical console errors

## Backend Tests:
```bash
# Test session creation
curl -X POST http://localhost:3001/api/sessions \
  -H "Content-Type: application/json" \
  -d '{"hostName":"Test Host","hostAvatar":"👨‍💻"}'

# Should return session with host as participant
```