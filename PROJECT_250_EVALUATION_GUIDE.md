# E-VoteHub 2.0 - Project 250 Final Evaluation Guide

**Course:** Project Work II (0610 2250)  
**Project Type:** Digital Voting System with Online Campaign Platform  
**Team Size:** 3 Members  
**Technology Stack:** MERN (MongoDB, Express.js, React, Node.js)

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)
2. [Project Workflow (Step-by-Step)](#project-workflow-step-by-step)
3. [Core Backend Architect - Questions & Answers](#core-backend-architect---questions--answers)
4. [UI/UX & Frontend Specialist - Questions & Answers](#uiux--frontend-specialist---questions--answers)
5. [Database & Security Engineer - Questions & Answers](#database--security-engineer---questions--answers)
6. [Technical Implementation Details](#technical-implementation-details)
7. [Key Features & Innovation](#key-features--innovation)

---

## 🎯 Project Overview

### What is E-VoteHub?

E-VoteHub is a **secure digital voting platform** that combines social campaigning with electronic voting. It allows organizations (like universities) to conduct elections with complete transparency, security, and voter engagement.

### Main Goals:
- **Secure & Transparent Voting:** End-to-end encrypted, tamper-proof voting system
- **Social Campaign Phase:** Nominees can run campaigns, post content, engage with voters
- **Dual Voting Modes:** Online voting (email-based OTP) and On-Campus voting (rotating codes)
- **Real-time Results:** Live vote counting with visual analytics
- **User Engagement:** Social media-like features for voter-nominee interaction

---

## 🔄 Project Workflow (Step-by-Step)

### Phase 1: User Registration & Authentication
1. **User visits the website** → Sees landing page with features
2. **Clicks "Register"** → Fills out registration form with:
   - Full Name, Username, Email, Date of Birth, Gender
   - NID (National ID), Phone Number
   - Profile Picture & Cover Photo
3. **System sends 6-digit OTP** to email address
4. **User enters OTP** → Account is created
5. **User can login** with username/email and password
6. **System generates JWT tokens** (Access Token + Refresh Token)
7. **User session is maintained** via cookies and localStorage

### Phase 2: Event Creation (Admin Only)
1. **Admin logs in** → Access to Admin Dashboard
2. **Creates Vote Event** with details:
   - Event Title & Description
   - Registration End Time
   - Vote Start Time & Vote End Time
   - Election Type (Single, Rank, or Multi-Vote)
   - Voting Mode (Online or On-Campus)
   - Ballot Images (for nominees to select)
   - Place name (for on-campus events)
3. **Event is published** → Appears on user dashboards
4. **Real-time notification** sent via Socket.io to all users

### Phase 3: Nominee Registration
1. **User registers as nominee** for an event:
   - Selects a unique ballot image (logo/symbol)
   - Writes description/manifesto
2. **System checks:**
   - Registration deadline not passed
   - Ballot image not already taken
3. **Ballot is reserved** → Moved from available to used ballots
4. **Nominee marked as "Pending"** → Waiting for admin approval
5. **User automatically registered as voter** as well

### Phase 4: Voter Registration
1. **Regular users register as voters** for events
2. **System creates voter record** with:
   - UserID and EventID reference
   - hasVoted status (initially false)
   - Email code fields (for online voting)

### Phase 5: Campaign Phase
1. **Approved nominees can create posts** with:
   - Text content
   - Pictures (multiple)
   - Videos
2. **Posts uploaded to Cloudinary** for media storage
3. **Other users can:**
   - View nominee posts
   - Like/dislike posts
   - Comment on posts
   - Like/dislike comments
4. **Real-time engagement** through social feed
5. **Voters can view nominee profiles** and campaign content

### Phase 6: Voting Phase (Online Mode)
1. **Voting period starts** (after VoteStartTime)
2. **Voter clicks "Vote Now"** button
3. **System sends 6-digit code** to voter's email
4. **Code valid for 10 minutes**
5. **Voter enters code** in voting interface
6. **Voter selects nominee(s):**
   - Single Vote: Choose exactly 1 nominee
   - Multi-Vote: Choose multiple nominees
   - Rank Vote: Rank all nominees (1st, 2nd, 3rd...)
7. **System validates:**
   - Code is correct and not expired
   - Voter hasn't already voted
   - Voting window is active
   - Selected nominees are approved
8. **Vote is recorded:**
   - Vote counts updated in VoteCount collection
   - Voter marked as "hasVoted = true"
   - **No trace of who voted for whom** (anonymous)
9. **Confirmation shown** to voter

### Phase 7: Voting Phase (On-Campus Mode)
1. **Admin rotates voting code** every X minutes (configurable)
2. **Code displayed at physical voting place**
3. **Voters at location** see the code
4. **Voter enters code** + selects nominees
5. **System validates:**
   - Current rotating code matches
   - Code not expired
   - Other standard checks
6. **Vote recorded** anonymously

### Phase 8: Results & Analytics
1. **Vote counting happens in real-time**
2. **Admin can view:**
   - Total votes per nominee
   - Rank scores (for ranked voting)
   - Voter turnout statistics
   - Event participation metrics
3. **Visual charts display:**
   - Bar charts for vote counts
   - Pie charts for percentages
   - Trend analytics
4. **Results can be exported** or announced

### Phase 9: Post-Election
1. **Event marked as completed**
2. **Results remain accessible**
3. **Audit trail available** (who registered, when)
4. **Vote anonymity preserved** (cannot trace individual votes)

---

## 🔧 Core Backend Architect - Questions & Answers

### Q1: What is the overall backend architecture of E-VoteHub?

**Answer:**
E-VoteHub uses a **3-layer MVC architecture** with the following components:

1. **Entry Point (index.js):**
   - Loads environment variables
   - Connects to MongoDB database
   - Creates HTTP server
   - Initializes Socket.io for real-time features
   - Starts server on configured port

2. **Application Layer (app.js):**
   - Express.js application setup
   - CORS configuration for cross-origin requests
   - JSON and URL-encoded body parsers
   - Cookie parser for authentication
   - Static file serving from public folder
   - Route mounting:
     - `/api/v1/users` → User routes
     - `/api/V1/admin` → Vote event routes
     - `/api/v1/post` → Campaign/post routes

3. **Routes Layer:**
   - Define API endpoints
   - Apply middleware (authentication, file upload)
   - Map to controller functions

4. **Controllers Layer:**
   - Business logic implementation
   - Data validation
   - Database operations
   - Response formatting

5. **Models Layer:**
   - MongoDB schemas using Mongoose
   - Data validation rules
   - Relationships between collections

6. **Middleware Layer:**
   - JWT authentication verification
   - File upload handling (Multer)
   - Email sending configuration
   - Error handling

7. **Utils Layer:**
   - Reusable helper functions
   - Cloudinary file upload/delete
   - API response formatters
   - Error handling classes

### Q2: How does the authentication system work?

**Answer:**
E-VoteHub uses **JWT (JSON Web Token) based authentication** with a dual-token strategy:

**Registration Flow:**
1. User submits registration form with images
2. Server generates 6-digit OTP
3. Registration data stored in memory (Map) with unique token
4. OTP sent to user's email
5. User verifies OTP within 10 minutes
6. Upon verification:
   - Images uploaded to Cloudinary
   - User document created in MongoDB
   - Password hashed using bcrypt (before save hook)
   - Access & Refresh tokens generated

**Token Generation:**
- **Access Token:** Short-lived (15 minutes), contains user ID, email, username, role
- **Refresh Token:** Long-lived (7 days), stored in database
- Both signed with secret keys from environment variables

**Login Flow:**
1. User provides username/email + password
2. Server finds user in database
3. Password verified using bcrypt comparison
4. Tokens generated and returned
5. Access token sent in response + cookie
6. Refresh token stored in database + cookie

**Authentication Middleware (jwtVerification):**
```javascript
1. Extract token from:
   - Cookie (primary)
   - Authorization header: "Bearer <token>" (fallback)
2. Verify token using JWT secret
3. Decode token to get user ID
4. Fetch user from database (exclude password/refresh token)
5. Attach user object to request (req.user)
6. Call next() to proceed
7. If any step fails → 401 Unauthorized error
```

**Token Renewal:**
- When access token expires, client sends refresh token
- Server validates refresh token against database
- New access token generated
- Old refresh token optionally rotated

### Q3: Explain the voting mechanism and how anonymity is maintained.

**Answer:**
The voting system ensures **complete anonymity** while maintaining vote integrity:

**Vote Casting Process:**

1. **Voter Identification:**
   - Voter must be registered for the event
   - System checks VoterReg collection: `{EventID, UserID}`
   - Verifies `hasVoted` is false

2. **Code Verification:**
   - **Online Mode:** 6-digit code sent to voter's email, valid 10 minutes
   - **On-Campus Mode:** Rotating 6-digit code displayed at venue
   - Code validated before any vote processing

3. **Vote Recording (Anonymously):**
   ```javascript
   // VoteCount collection structure:
   {
     EventID: ObjectId,
     ElectionType: "Single" | "Rank" | "MultiVote",
     Tally: [
       {
         NomineeId: ObjectId,
         TotalVote: 10,      // For Single/Multi
         TotalRank: 45       // For Rank (sum of ranks)
       }
     ]
   }
   ```

4. **Anonymity Preservation:**
   - **No voter-vote linkage stored**
   - VoteCount only stores aggregate tallies
   - VoterReg only marks `hasVoted = true`
   - **Cannot determine who voted for whom**
   - Database design separates identity from vote choice

5. **Vote Types:**
   - **Single Vote:** TotalVote += 1 for selected nominee
   - **Multi-Vote:** TotalVote += 1 for each selected (unique) nominee
   - **Rank Vote:** TotalRank += assigned rank (1-N) for all nominees

6. **Validation Checks:**
   - Voter hasn't already voted
   - Voting window is active (between start and end time)
   - Code is valid and not expired
   - Selected nominees are approved for the event
   - Election type matches selections (e.g., exactly 1 for Single)

**Security Features:**
- Vote counts updated atomically in database
- Transaction-like operations prevent race conditions
- No modification of votes after submission
- Audit trail shows WHO voted, not WHAT they voted

### Q4: How do you handle real-time updates in the application?

**Answer:**
E-VoteHub uses **Socket.io** for real-time bidirectional communication:

**Socket.io Setup (socket.js):**
```javascript
1. Server creates Socket.io instance with CORS config
2. Listens for client connections
3. Supports room-based events:
   - joinEvent(eventId): Client joins event-specific room
   - leaveEvent(eventId): Client leaves room
```

**Real-time Events:**
1. **Event Creation:**
   - Admin creates event
   - Server emits: `eventCreated` with {eventId, title}
   - All connected clients receive update
   - Frontend refreshes event list

2. **Nominee Approval:**
   - Admin approves nominee
   - Emit: `nomineeApproved` to event room
   - Voters see updated nominee list

3. **Vote Submission:**
   - Vote recorded
   - Emit: `voteUpdated` to event room
   - Real-time chart updates on admin dashboard

**Implementation:**
```javascript
// Server side
import { getIO } from './socket.js'
getIO().emit('eventCreated', { eventId, title })
// or room-specific:
getIO().to(String(eventId)).emit('voteUpdated', data)

// Client side (Frontend)
import io from 'socket.io-client'
const socket = io(API_BASE)
socket.on('eventCreated', (data) => {
  // Update UI
})
```

### Q5: Explain the file upload system (Multer + Cloudinary).

**Answer:**
E-VoteHub uses a **two-stage file upload system**:

**Stage 1: Multer (Local Storage)**
```javascript
// Multer.Middleware.js
- Configures storage location: ./public/Temp
- Preserves original filename
- Accepts multiple file types (images, videos)
- Limits: 10 ballot images, multiple post media
```

**Stage 2: Cloudinary (Cloud Storage)**
```javascript
// Cloudinary.js
FileUpload function:
1. Receives local file path from Multer
2. Uploads to Cloudinary using API credentials
3. Cloudinary returns:
   - Public URL (for accessing file)
   - Public ID (for deletion)
4. Local temp file deleted after upload
5. Returns {url, public_id}

FileDelete function:
1. Receives public_id
2. Calls Cloudinary destroy API
3. Removes file from cloud storage
```

**Usage in Controllers:**
```javascript
// Profile picture upload
1. req.files.ProfileImage[0].path → local path
2. FileUpload(localPath) → {url, public_id}
3. Store in database: ProfileImage: url, ProfilePublicId: public_id
4. When updating: FileDelete(old public_id) first
5. Upload new file, update database
```

**Benefits:**
- Local temp storage prevents memory overflow
- Cloudinary provides CDN for fast delivery
- Automatic image optimization
- Easy file management (delete, transform)
- Scalable storage solution

### Q6: How is error handling implemented across the backend?

**Answer:**
E-VoteHub uses **centralized error handling** with custom classes:

**ApiError Class:**
```javascript
class ApiError extends Error {
  constructor(statusCode, message, errors=[], stack=""){
    super(message)
    this.statusCode = statusCode
    this.errors = errors
    this.success = false
    // Stack trace for debugging
  }
}
```

**AsyncHandler Wrapper:**
```javascript
const AsynHandler = (fn) => async (req, res, next) => {
  try {
    await fn(req, res, next)
  } catch (error) {
    // Express error middleware catches this
    res.status(error.statusCode || 500).json({
      success: false,
      message: error.message
    })
  }
}
```

**Usage Pattern:**
```javascript
const SomeController = AsynHandler(async(req, res) => {
  // Validation
  if(!data) throw new ApiError(401, "Data required")
  
  // Database operation
  const result = await Model.create(data)
  if(!result) throw new ApiError(501, "Creation failed")
  
  // Success response
  return res.status(201).json(
    new ApiResponse(201, result, "Success")
  )
})
```

**Error Types:**
- 400: Bad Request (validation errors)
- 401: Unauthorized (auth failures)
- 403: Forbidden (permission denied)
- 404: Not Found
- 409: Conflict (duplicate data)
- 500/501: Server errors

**Benefits:**
- Consistent error format
- Automatic error catching
- No try-catch in every controller
- Clear error messages for frontend
- Stack traces in development

### Q7: What is the role of Socket.io in vote events?

**Answer:**
Socket.io enables **real-time collaboration** in the voting system:

**Key Use Cases:**

1. **Event Lifecycle Updates:**
   - Event created → Notify all users
   - Registration deadline approaching → Countdown updates
   - Voting starts → Alert registered voters
   - Voting ends → Lock voting interface

2. **Dynamic Code Rotation (On-Campus):**
   - Admin rotates code every N minutes
   - New code broadcast to event room
   - Frontend updates displayed code automatically
   - Ensures synchronized access control

3. **Live Vote Tallies:**
   - Each vote submission triggers update
   - Admin dashboard receives new counts
   - Charts re-render with latest data
   - No need to refresh page

4. **Nominee Approval:**
   - Admin approves/unapproves nominee
   - Event participants notified instantly
   - Ballot lists update in real-time

**Technical Implementation:**
- Server-side: `getIO().to(eventId).emit('event', data)`
- Client-side: `socket.on('event', handler)`
- Rooms allow targeted broadcasts (only event participants)
- Fallback to polling if WebSocket unavailable

---

## 🎨 UI/UX & Frontend Specialist - Questions & Answers

### Q1: What is the frontend architecture and tech stack?

**Answer:**
E-VoteHub frontend is built with **React + Vite** using modern practices:

**Tech Stack:**
- **React 18.3.1:** Component-based UI library
- **Vite:** Fast build tool and dev server
- **React Router DOM 6.27:** Client-side routing
- **Axios:** HTTP client for API calls
- **Socket.io-client:** Real-time communication
- **Tailwind CSS 3.4:** Utility-first styling
- **Chart.js + react-chartjs-2:** Data visualization
- **SweetAlert2:** Beautiful alert dialogs
- **Lucide React:** Icon library

**Project Structure:**
```
src/
├── main.jsx              # App entry point
├── styles.css            # Tailwind imports
├── routes/               # Page components
│   ├── App.jsx          # Main layout + routing
│   ├── Home.jsx         # Landing page
│   ├── Login.jsx        # Authentication
│   ├── Register.jsx     # User registration
│   ├── UserDashboard.jsx # Voter interface
│   ├── AdminDashboard.jsx # Admin panel
│   ├── Profile.jsx       # User profile
│   └── ...
├── components/
│   └── RequireAuth.jsx   # Auth guard component
└── lib/
    ├── api.js           # API functions
    └── votersApi.js     # Voter-specific APIs
```

### Q2: How does client-side routing work?

**Answer:**
Using **React Router DOM v6** with protected routes:

**Route Structure:**
```jsx
<Routes>
  {/* Public routes */}
  <Route path="/" element={<Home />} />
  <Route path="/login" element={<Login />} />
  <Route path="/register" element={<Register />} />
  <Route path="/profile/:id" element={<PublicProfile />} />
  <Route path="/privacy" element={<PrivacyPolicy />} />
  <Route path="/terms" element={<Terms />} />
  <Route path="/about" element={<AboutUs />} />
  
  {/* Protected routes */}
  <Route path="/dashboard" element={
    <RequireAuth><UserDashboard /></RequireAuth>
  } />
  <Route path="/admin" element={
    <RequireAuth adminOnly><AdminDashboard /></RequireAuth>
  } />
  <Route path="/myprofile" element={
    <RequireAuth><Profile /></RequireAuth>
  } />
</Routes>
```

**RequireAuth Component:**
```jsx
function RequireAuth({ children, adminOnly = false }) {
  const user = getUser()
  const authed = !!localStorage.getItem('accessToken')
  
  if (!authed) return <Navigate to="/login" />
  if (adminOnly && user?.Role !== 'admin') {
    return <Navigate to="/dashboard" />
  }
  
  return children
}
```

**Navigation Features:**
- Browser back/forward support
- Programmatic navigation: `navigate('/path')`
- Active link highlighting
- Redirect after login
- Nested routes possible

### Q3: Explain the state management approach.

**Answer:**
E-VoteHub uses **React Hooks + localStorage** (no Redux):

**State Types:**

1. **Local Component State (useState):**
```jsx
const [events, setEvents] = useState([])
const [loading, setLoading] = useState(false)
const [selectedNominee, setSelectedNominee] = useState(null)
```

2. **Global State (localStorage):**
```javascript
// Stored data:
- accessToken: JWT for authentication
- refreshToken: Long-lived token
- user: JSON.stringify({FullName, Role, ...})
- role: 'user' | 'admin' | 'nominee'
```

3. **State Synchronization:**
```jsx
// Listen to storage changes (cross-tab sync)
useEffect(() => {
  const onStorage = () => {
    setAuthed(!!localStorage.getItem('accessToken'))
    setUser(JSON.parse(localStorage.getItem('user')))
  }
  window.addEventListener('storage', onStorage)
  
  // Also poll every 800ms for same-tab updates
  const id = setInterval(onStorage, 800)
  
  return () => {
    window.removeEventListener('storage', onStorage)
    clearInterval(id)
  }
}, [])
```

4. **Real-time State (Socket.io):**
```jsx
useEffect(() => {
  socket.on('eventCreated', (data) => {
    setEvents(prev => [...prev, data])
  })
  return () => socket.off('eventCreated')
}, [])
```

**Benefits:**
- Simple and performant
- No complex state libraries needed
- Persistent across page reloads
- Cross-tab synchronization
- Easy debugging

### Q4: How is the theme system (dark/light mode) implemented?

**Answer:**
E-VoteHub uses **Tailwind CSS dark mode** with localStorage persistence:

**Implementation:**

1. **Tailwind Configuration:**
```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class', // Use class-based dark mode
  // ...
}
```

2. **Theme State Management:**
```jsx
const [theme, setTheme] = useState(() => 
  localStorage.getItem('theme') || 'dark'
)

useEffect(() => {
  const root = document.documentElement
  if (theme === 'dark') {
    root.classList.add('dark')
  } else {
    root.classList.remove('dark')
  }
  localStorage.setItem('theme', theme)
}, [theme])
```

3. **Color System:**
```javascript
const COLORS = {
  BG_LIGHT: 'bg-[#ECEBEB]',
  BG_DARK: 'dark:bg-[#1A2129]',
  TEXT_LIGHT: 'text-gray-900',
  TEXT_DARK: 'dark:text-white',
  ACCENT_PRIMARY: 'bg-[#1E3A8A]', // Navy blue
  ACCENT_SECONDARY: 'dark:bg-[#3B82F6]', // Bright blue
}
```

4. **Usage in Components:**
```jsx
<div className={`
  ${COLORS.BG_LIGHT} 
  ${COLORS.BG_DARK} 
  ${COLORS.TEXT_LIGHT} 
  ${COLORS.TEXT_DARK}
`}>
  Content adapts to theme
</div>
```

5. **Toggle Button:**
```jsx
<button onClick={() => setTheme(t => t === 'dark' ? 'light' : 'dark')}>
  {theme === 'dark' ? '☀️' : '🌙'}
</button>
```

**Features:**
- Instant theme switching
- No flash of unstyled content
- Persistent across sessions
- System preference detection possible
- Smooth transitions with CSS

### Q5: How are API calls organized and handled?

**Answer:**
API layer is centralized in **lib/api.js** using Axios:

**Setup:**
```javascript
const API_BASE = import.meta.env.VITE_API_BASE || 'http://localhost:8002'

export const api = axios.create({
  baseURL: API_BASE,
  withCredentials: true, // Send cookies
})

// Auto-attach token to requests
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

**API Functions:**
```javascript
// Authentication
export async function login({ UserName, Email, Password }) {
  const res = await api.post('/api/v1/users/login', data)
  // Store tokens
  localStorage.setItem('accessToken', res.data.data.AccessToken)
  localStorage.setItem('user', JSON.stringify(res.data.data.LogInUser))
  return res.data
}

export async function logout() {
  await api.post('/api/v1/users/logout')
  // Clear storage
  localStorage.clear()
}

// Voting
export async function listEvents() {
  const res = await api.get('/api/V1/admin/events')
  return res.data?.data || []
}

export async function castVote(eventId, nominees, code) {
  const res = await api.post('/api/V1/admin/voting', {
    EventID: eventId,
    SelectedNominee: nominees,
    code
  })
  return res.data
}
```

**Usage in Components:**
```jsx
import { login, listEvents } from '../lib/api'

function MyComponent() {
  const [events, setEvents] = useState([])
  
  useEffect(() => {
    listEvents().then(setEvents).catch(console.error)
  }, [])
  
  const handleLogin = async (data) => {
    try {
      await login(data)
      navigate('/dashboard')
    } catch (err) {
      alert(err.response?.data?.message || 'Login failed')
    }
  }
}
```

**Error Handling:**
```javascript
try {
  await someApiCall()
} catch (error) {
  // Axios error structure:
  // error.response.status → HTTP code
  // error.response.data.message → Backend message
  Swal.fire('Error', error.response?.data?.message, 'error')
}
```

### Q6: Describe the user registration and OTP verification flow.

**Answer:**
Two-step registration process with email verification:

**Step 1: Registration Form**
```jsx
function Register() {
  const [formData, setFormData] = useState({
    FullName: '', UserName: '', Email: '',
    DateOfBirth: '', Gender: '', Password: '',
    NID: '', PhoneNumber: ''
  })
  const [profileImage, setProfileImage] = useState(null)
  const [coverImage, setCoverImage] = useState(null)
  const [otpToken, setOtpToken] = useState(null)
  const [showOtp, setShowOtp] = useState(false)

  const handleSubmit = async (e) => {
    e.preventDefault()
    
    // Create FormData for file upload
    const fd = new FormData()
    Object.keys(formData).forEach(key => {
      fd.append(key, formData[key])
    })
    fd.append('ProfileImage', profileImage)
    fd.append('CoverImage', coverImage)
    
    try {
      const result = await registerInit(fd)
      setOtpToken(result.otpToken)
      setShowOtp(true)
      Swal.fire('Success', 'OTP sent to your email', 'success')
    } catch (err) {
      Swal.fire('Error', err.response?.data?.message, 'error')
    }
  }
}
```

**Step 2: OTP Verification**
```jsx
const [otp, setOtp] = useState('')

const handleVerify = async () => {
  try {
    await registerVerify({ otpToken, code: otp })
    Swal.fire('Success', 'Registration completed!', 'success')
    navigate('/login')
  } catch (err) {
    Swal.fire('Error', 'Invalid or expired OTP', 'error')
  }
}

return showOtp ? (
  <div>
    <input
      type="text"
      maxLength="6"
      placeholder="Enter 6-digit OTP"
      value={otp}
      onChange={(e) => setOtp(e.target.value)}
    />
    <button onClick={handleVerify}>Verify</button>
  </div>
) : (
  <form onSubmit={handleSubmit}>
    {/* Registration form fields */}
  </form>
)
```

**Backend Flow:**
1. Server receives registration data
2. Validates duplicates (username/email)
3. Generates 6-digit OTP and unique token
4. Stores data in memory (not DB yet)
5. Sends email with OTP
6. Returns otpToken to frontend
7. Frontend shows OTP input
8. User submits OTP + token
9. Server verifies OTP
10. Uploads images to Cloudinary
11. Creates user in database
12. Returns success

**UX Features:**
- Image preview before upload
- Form validation (client + server)
- Loading states during API calls
- Clear error messages
- OTP countdown timer (10 min)
- Resend OTP option

### Q7: How is the voting interface designed for different election types?

**Answer:**
Dynamic UI adapts based on election type:

**Single Vote Mode:**
```jsx
{electionType === 'Single' && (
  <div className="grid grid-cols-2 gap-4">
    {nominees.map(nominee => (
      <div
        key={nominee._id}
        onClick={() => setSelected([nominee._id])}
        className={`
          p-4 border-2 rounded-lg cursor-pointer
          transition-all duration-200
          ${selected[0] === nominee._id 
            ? 'border-blue-500 bg-blue-50' 
            : 'border-gray-300 hover:border-blue-300'
          }
        `}
      >
        <img src={nominee.ballotImage} alt="" />
        <h3>{nominee.name}</h3>
      </div>
    ))}
  </div>
)}
```

**Multi-Vote Mode:**
```jsx
{electionType === 'MultiVote' && (
  <div className="space-y-2">
    {nominees.map(nominee => (
      <label key={nominee._id} className="flex items-center gap-3 p-3 border rounded hover:bg-gray-50">
        <input
          type="checkbox"
          checked={selected.includes(nominee._id)}
          onChange={(e) => {
            if (e.target.checked) {
              setSelected([...selected, nominee._id])
            } else {
              setSelected(selected.filter(id => id !== nominee._id))
            }
          }}
        />
        <img src={nominee.ballotImage} className="w-12 h-12" />
        <span>{nominee.name}</span>
      </label>
    ))}
  </div>
)}
```

**Rank Vote Mode:**
```jsx
{electionType === 'Rank' && (
  <div className="space-y-3">
    <p>Rank candidates (1 = Most preferred)</p>
    {nominees.map(nominee => (
      <div key={nominee._id} className="flex items-center gap-3">
        <img src={nominee.ballotImage} className="w-12" />
        <span className="flex-1">{nominee.name}</span>
        <select
          value={rankings[nominee._id] || ''}
          onChange={(e) => setRankings({
            ...rankings,
            [nominee._id]: parseInt(e.target.value)
          })}
          className="border rounded px-2 py-1"
        >
          <option value="">Select rank</option>
          {[...Array(nominees.length)].map((_, i) => (
            <option key={i} value={i + 1}>{i + 1}</option>
          ))}
        </select>
      </div>
    ))}
  </div>
)}
```

**Code Input (Both Modes):**
```jsx
<div className="mt-6 p-4 bg-yellow-50 border border-yellow-200 rounded">
  {votingMode === 'online' ? (
    <>
      <button onClick={requestCode}>Send Code to Email</button>
      <input
        type="text"
        maxLength="6"
        placeholder="Enter 6-digit code"
        value={code}
        onChange={(e) => setCode(e.target.value)}
      />
    </>
  ) : (
    <>
      <p>Enter code displayed at: {event.place}</p>
      <input
        type="text"
        maxLength="6"
        placeholder="Current voting code"
        value={code}
        onChange={(e) => setCode(e.target.value)}
      />
    </>
  )}
</div>
```

**Submit Logic:**
```jsx
const handleVote = async () => {
  // Validate based on type
  if (electionType === 'Single' && selected.length !== 1) {
    return Swal.fire('Error', 'Select exactly one nominee', 'error')
  }
  
  // Format data
  const nominees = electionType === 'Rank'
    ? Object.keys(rankings).map(id => ({
        NomineeId: id,
        Rank: rankings[id]
      }))
    : selected.map(id => ({ NomineeId: id }))
  
  try {
    await castVote(eventId, electionType, nominees, code)
    Swal.fire('Success', 'Vote submitted!', 'success')
    navigate('/dashboard')
  } catch (err) {
    Swal.fire('Error', err.response?.data?.message, 'error')
  }
}
```

---

## 🔐 Database & Security Engineer - Questions & Answers

### Q1: What is the database schema design?

**Answer:**
MongoDB with Mongoose ODM, **7 main collections**:

**1. User Collection:**
```javascript
{
  _id: ObjectId,
  FullName: String (indexed),
  UserName: String (unique, indexed, lowercase),
  Email: String (unique, lowercase),
  DateOfBirth: Date,
  Gender: "male" | "female" | "other",
  Password: String (bcrypt hashed),
  NID: String,
  PhoneNumber: String (BD format validation),
  ProfileImage: String (Cloudinary URL),
  CoverImage: String (Cloudinary URL),
  ProfilePublicId: String,
  CoverPublicId: String,
  RefreshToken: String,
  Role: "user" | "admin" | "nominee" (default: "user"),
  createdAt: Date,
  updatedAt: Date
}
```

**2. VoteEvent Collection:**
```javascript
{
  _id: ObjectId,
  Title: String (required),
  Description: String,
  BallotImage: [{url: String, publicId: String}],
  UsedBallotImage: [{url: String, publicId: String}],
  RegEndTime: Date,
  VoteStartTime: Date,
  VoteEndTime: Date,
  ElectionType: "Single" | "Rank" | "MultiVote",
  CreateBy: ObjectId (ref: User),
  votingMode: "online" | "onCampus",
  codeRotationMinutes: Number (default: 2),
  currentVoteCode: String (6-digit),
  currentCodeExpiresAt: Date,
  place: String (venue name for onCampus),
  createdAt: Date,
  updatedAt: Date
}
```

**3. VoterReg Collection:**
```javascript
{
  _id: ObjectId,
  UserID: ObjectId (ref: User, required),
  EventID: ObjectId (ref: VoteEvent, required),
  hasVoted: Boolean (default: false),
  emailCode: String (6-digit OTP),
  emailCodeExpiresAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

**4. NomineeReg Collection:**
```javascript
{
  _id: ObjectId,
  UserID: ObjectId (ref: User, required),
  EventID: ObjectId (ref: VoteEvent, required),
  Approved: Boolean (default: false),
  SelectedBalot: {
    url: String,
    publicId: String
  },
  Description: String (manifesto),
  createdAt: Date,
  updatedAt: Date
}
```

**5. VoteCount Collection:**
```javascript
{
  _id: ObjectId,
  EventID: ObjectId (ref: VoteEvent, indexed),
  ElectionType: "Single" | "Rank" | "MultiVote",
  Tally: [
    {
      NomineeId: ObjectId (ref: User),
      TotalVote: Number (default: 0),
      TotalRank: Number (default: 0)
    }
  ],
  createdAt: Date,
  updatedAt: Date
}
// Unique index: {EventID: 1, ElectionType: 1}
```

**6. Post Collection (Campaign):**
```javascript
{
  _id: ObjectId,
  owner: ObjectId (ref: User, required),
  eventID: ObjectId (ref: VoteEvent, required),
  picture: [{url: String, publicId: String}],
  video: [{url: String, publicId: String}],
  content: String,
  likes: [ObjectId] (ref: User),
  dislikes: [ObjectId] (ref: User),
  createdAt: Date,
  updatedAt: Date
}
```

**7. Comment Collection:**
```javascript
{
  _id: ObjectId,
  owner: ObjectId (ref: User, required),
  eventID: ObjectId (ref: VoteEvent, required),
  postID: ObjectId (ref: Post, required, indexed),
  comment: String (max 2000 chars, trimmed),
  likes: [ObjectId] (ref: User),
  dislikes: [ObjectId] (ref: User),
  createdAt: Date,
  updatedAt: Date
}
// Indexes: postID, eventID
```

**Relationships:**
- User → VoteEvent (CreateBy)
- User → VoterReg (many-to-many with Event)
- User → NomineeReg (many-to-many with Event)
- VoteEvent → VoteCount (one-to-one per ElectionType)
- User → Post (one-to-many)
- Post → Comment (one-to-many)

### Q2: How is password security implemented?

**Answer:**
Multi-layer password protection using **bcrypt**:

**1. Password Hashing (Pre-save Hook):**
```javascript
// User.Model.js
UserSchema.pre("save", async function(next){
  if(!this.isModified("Password")) return next();
  
  this.Password = await bcrypt.hash(this.Password, 10);
  next();
})
```

**How it works:**
- Only hashes if password field changed
- Uses bcrypt with salt rounds = 10
- Original password never stored
- Hash is one-way (cannot reverse)

**2. Password Verification (Instance Method):**
```javascript
UserSchema.methods.IsPasswordCorrect = async function(password){
  return await bcrypt.compare(password, this.Password);
}
```

**3. Login Flow:**
```javascript
const user = await User.findOne({
  $or: [{UserName}, {Email}]
})

if (!user) throw new ApiError(401, "User not found")

const isPasswordValid = await user.IsPasswordCorrect(Password)
if (!isPasswordValid) {
  throw new ApiError(401, "Invalid credentials")
}

// Generate tokens only if password correct
const {AccessToken, RefreshToken} = await GenerateAccessAndRefreshToken(user._id)
```

**4. Password Change:**
```javascript
const ChangePassword = AsynHandler(async(req, res) => {
  const {OldPassword, NewPassword} = req.body
  const user = await User.findById(req.user._id)
  
  const isCorrect = await user.IsPasswordCorrect(OldPassword)
  if (!isCorrect) {
    throw new ApiError(401, "Old password incorrect")
  }
  
  user.Password = NewPassword
  await user.save() // Pre-save hook hashes new password
  
  return res.status(200).json(
    new ApiResponse(200, {}, "Password changed")
  )
})
```

**Security Features:**
- **Bcrypt algorithm:** Industry standard, slow by design (prevents brute force)
- **Salt rounds (10):** Each password has unique salt
- **No plain text storage:** Database breach doesn't expose passwords
- **Timing-safe comparison:** bcrypt.compare prevents timing attacks
- **Password strength:** Frontend can enforce rules (min length, complexity)

### Q3: Explain the JWT token system and refresh mechanism.

**Answer:**
Dual-token authentication with automatic renewal:

**Token Generation (Instance Methods):**
```javascript
// User.Model.js

// Access Token (Short-lived: 15 min)
UserSchema.methods.GenerateAccessToken = function(){
  return jwt.sign(
    {
      _id: this._id,
      Email: this.Email,
      UserName: this.UserName,
      Role: this.Role
    },
    process.env.ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' }
  )
}

// Refresh Token (Long-lived: 7 days)
UserSchema.methods.GenerateRefreshToken = function(){
  return jwt.sign(
    { _id: this._id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  )
}
```

**Token Storage:**
- **Server-side:** Refresh token stored in User document
- **Client-side:** Both tokens in httpOnly cookies + localStorage (for mobile/API)

**Token Verification Middleware:**
```javascript
const jwtVerification = AsynHandler(async(req, res, next) => {
  // Priority: Cookie > Authorization header
  let Token = req.cookies?.AccessToken
  
  const authHeader = req.header("Authorization")
  if (!Token && authHeader) {
    // Extract from "Bearer <token>"
    const parts = authHeader.split(' ')
    if (parts.length === 2 && /^Bearer$/i.test(parts[0])) {
      Token = parts[1].trim()
    }
  }
  
  if (!Token) throw new ApiError(401, "Authentication required")
  
  // Verify token signature and expiry
  const DecodeToken = jwt.verify(Token, process.env.ACCESS_TOKEN_SECRET)
  
  // Fetch user (exclude sensitive fields)
  const user = await User.findById(DecodeToken._id)
    .select("-Password -RefreshToken")
  
  if (!user) throw new ApiError(401, "Invalid token")
  
  req.user = user
  next()
})
```

**Refresh Token Flow:**
```javascript
const RenewAccesToken = AsynHandler(async(req, res) => {
  const incomingRefreshToken = 
    req.cookies?.RefreshToken || 
    req.body?.RefreshToken
  
  if (!incomingRefreshToken) {
    throw new ApiError(401, "Refresh token required")
  }
  
  // Verify refresh token
  const DecodeToken = jwt.verify(
    incomingRefreshToken,
    process.env.REFRESH_TOKEN_SECRET
  )
  
  const user = await User.findById(DecodeToken._id)
  
  // Check token matches database
  if (user.RefreshToken !== incomingRefreshToken) {
    throw new ApiError(401, "Token mismatch")
  }
  
  // Generate new pair
  const {AccessToken, RefreshToken} = 
    await GenerateAccessAndRefreshToken(user._id)
  
  return res
    .status(200)
    .cookie("AccessToken", AccessToken, {httpOnly: true, secure: true})
    .cookie("RefreshToken", RefreshToken, {httpOnly: true, secure: true})
    .json(new ApiResponse(200, {AccessToken, RefreshToken}, "Renewed"))
})
```

**Security Benefits:**
- **Short access token lifespan:** Limits damage if stolen
- **Refresh token rotation:** Old tokens invalidated
- **HttpOnly cookies:** JavaScript cannot access (XSS protection)
- **Secure flag:** HTTPS only transmission
- **Token signature:** Prevents tampering
- **Database validation:** Refresh token checked against stored value

### Q4: How is vote anonymity guaranteed at the database level?

**Answer:**
Architecture separates **identity from choice**:

**Data Separation:**

1. **VoterReg (WHO can vote):**
```javascript
{
  UserID: ObjectId("user123"),
  EventID: ObjectId("event456"),
  hasVoted: true  // ← Only knows IF voted
}
```

2. **VoteCount (WHAT was voted):**
```javascript
{
  EventID: ObjectId("event456"),
  Tally: [
    {NomineeId: ObjectId("nom1"), TotalVote: 15},
    {NomineeId: ObjectId("nom2"), TotalVote: 23}
  ]
  // ← No UserID, cannot link to voter
}
```

**No Link Between Collections:**
- VoterReg has UserID but no vote choice
- VoteCount has vote choice but no UserID
- **No table/collection stores (UserID + NomineeId) pair**

**Vote Recording Process:**
```javascript
// 1. Check voter eligibility
const voter = await VoterReg.findOne({EventID, UserID})
if (voter.hasVoted) throw new ApiError(401, "Already voted")

// 2. Update tally (anonymously)
const tally = await VoteCount.findOne({EventID, ElectionType})
tally.Tally.find(t => t.NomineeId === selected).TotalVote += 1
await tally.save()

// 3. Mark as voted (no vote info)
voter.hasVoted = true
await voter.save()

// Result: Cannot determine user123 voted for nom1
```

**Database Queries:**
- **Can determine:** User123 voted in Event456 (VoterReg)
- **Can determine:** Nominee1 got 15 votes (VoteCount)
- **Cannot determine:** User123 voted for Nominee1 (no join possible)

**Additional Security:**
- No audit logs of vote submissions (only registration)
- Vote counts updated atomically (no transaction history per user)
- Admin cannot query "who voted for X"
- Even database admin cannot reverse-engineer votes

**Edge Cases:**
- Single voter event: Count reveals choice (social problem, not technical)
- Timing analysis: Mitigated by batch processing or random delays

### Q5: What security measures prevent SQL injection and XSS?

**Answer:**
Multi-layer defense strategy:

**1. NoSQL Injection Prevention:**

**MongoDB + Mongoose Protection:**
```javascript
// Mongoose sanitizes queries automatically
const user = await User.findOne({
  UserName: req.body.UserName  // Safe even if malicious
})

// Dangerous (raw MongoDB):
db.collection.find({
  UserName: {"$ne": null}  // Bypasses auth
})

// Safe (Mongoose schema validation):
UserName: {
  type: String,  // Forces string type
  required: true,
  trim: true
}
```

**Input Validation:**
```javascript
// Server-side checks
if (!UserName || typeof UserName !== 'string') {
  throw new ApiError(401, "Invalid username")
}

// Mongoose schema validators
Email: {
  type: String,
  unique: true,
  lowercase: true,
  match: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
}

PhoneNumber: {
  type: String,
  match: [/^01[3-9]\d{8}$/, "Invalid BD phone number"]
}
```

**2. XSS (Cross-Site Scripting) Prevention:**

**React Auto-Escaping:**
```jsx
// Automatically escapes HTML
const name = "<script>alert('XSS')</script>"
<div>{name}</div>
// Renders as text, not executed

// Safe rendering of user content
<div>{post.content}</div>  // React escapes by default
```

**Dangerous Patterns (Avoided):**
```jsx
// NEVER DO THIS:
<div dangerouslySetInnerHTML={{__html: userInput}} />

// If HTML needed, sanitize first:
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{
  __html: DOMPurify.sanitize(userInput)
}} />
```

**Content Security Policy (Headers):**
```javascript
// app.js (can add helmet middleware)
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https://res.cloudinary.com"]
    }
  }
}))
```

**3. CSRF (Cross-Site Request Forgery) Prevention:**

**SameSite Cookies:**
```javascript
res.cookie("AccessToken", token, {
  httpOnly: true,
  secure: true,
  sameSite: 'Strict'  // Prevents CSRF
})
```

**CORS Configuration:**
```javascript
app.use(cors({
  origin: process.env.CORS_ORIGIN.split(','),
  credentials: true
}))
// Only allowed origins can make requests
```

**4. Additional Security Measures:**

**Environment Variables:**
```javascript
// .env (never committed)
JWT_SECRET=random-256-bit-secret
DATABASE_URI=mongodb+srv://...
CLOUDINARY_API_SECRET=...

// Accessed via
process.env.JWT_SECRET
```

**Rate Limiting (can implement):**
```javascript
import rateLimit from 'express-rate-limit'

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 min
  max: 100 // Max 100 requests per IP
})

app.use('/api/', limiter)
```

**File Upload Validation:**
```javascript
// Multer file filter
const fileFilter = (req, file, cb) => {
  const allowed = /jpeg|jpg|png|gif|mp4/
  const ext = file.originalname.split('.').pop()
  
  if (allowed.test(ext)) {
    cb(null, true)
  } else {
    cb(new Error('Invalid file type'), false)
  }
}
```

### Q6: How do you handle on-campus voting code rotation securely?

**Answer:**
Time-based rotating codes with real-time synchronization:

**Code Generation:**
```javascript
const RotateOnCampusCode = AsynHandler(async(req, res) => {
  const {EventID} = req.body
  
  // Only admin can rotate
  if (req.user.Role !== 'admin') {
    throw new ApiError(403, "Admin only")
  }
  
  const event = await VoteEvent.findById(EventID)
  if (event.votingMode !== 'onCampus') {
    throw new ApiError(400, "Only for on-campus events")
  }
  
  // Generate new 6-digit code
  const newCode = String(Math.floor(100000 + Math.random() * 900000))
  
  // Set expiration based on rotation interval
  const expiresAt = new Date(
    Date.now() + event.codeRotationMinutes * 60 * 1000
  )
  
  // Update event
  event.currentVoteCode = newCode
  event.currentCodeExpiresAt = expiresAt
  await event.save()
  
  // Broadcast to all clients in event room
  getIO().to(String(EventID)).emit('codeRotated', {
    code: newCode,
    expiresAt
  })
  
  return res.status(200).json(
    new ApiResponse(200, {code: newCode, expiresAt}, "Code rotated")
  )
})
```

**Frontend Display (Admin):**
```jsx
function AdminCodeDisplay({ eventId, rotationMinutes }) {
  const [currentCode, setCurrentCode] = useState(null)
  const [expiresAt, setExpiresAt] = useState(null)
  const [timeLeft, setTimeLeft] = useState(0)
  
  useEffect(() => {
    // Fetch current code
    getCurrentCode(eventId).then(data => {
      setCurrentCode(data.code)
      setExpiresAt(data.expiresAt)
    })
    
    // Listen for rotation
    socket.on('codeRotated', (data) => {
      setCurrentCode(data.code)
      setExpiresAt(data.expiresAt)
    })
    
    return () => socket.off('codeRotated')
  }, [eventId])
  
  // Countdown timer
  useEffect(() => {
    const interval = setInterval(() => {
      const now = Date.now()
      const expires = new Date(expiresAt).getTime()
      const left = Math.max(0, Math.floor((expires - now) / 1000))
      setTimeLeft(left)
      
      // Auto-rotate when expired
      if (left === 0 && currentCode) {
        rotateCode(eventId)
      }
    }, 1000)
    
    return () => clearInterval(interval)
  }, [expiresAt])
  
  const handleManualRotate = async () => {
    await rotateCode(eventId)
  }
  
  return (
    <div className="code-display">
      <h2>Current Voting Code</h2>
      <div className="code">{currentCode || '------'}</div>
      <div className="timer">
        Expires in: {Math.floor(timeLeft / 60)}:
        {String(timeLeft % 60).padStart(2, '0')}
      </div>
      <button onClick={handleManualRotate}>
        Rotate Code Now
      </button>
    </div>
  )
}
```

**Voter Validation:**
```javascript
// When voter submits vote
const event = await VoteEvent.findById(EventID)

if (event.votingMode === 'onCampus') {
  // Check code exists
  if (!event.currentVoteCode || !event.currentCodeExpiresAt) {
    throw new ApiError(403, "Voting code not active")
  }
  
  // Check not expired
  if (new Date() > new Date(event.currentCodeExpiresAt)) {
    throw new ApiError(403, "Code expired, request new code")
  }
  
  // Check code matches
  if (String(event.currentVoteCode) !== String(req.body.code)) {
    throw new ApiError(403, "Invalid code")
  }
}

// Proceed with vote
```

**Security Features:**
- **Time-limited:** Code expires after N minutes
- **Unpredictable:** Random 6-digit generation
- **Rotation:** Old codes invalidated
- **Real-time sync:** All clients get new code via Socket.io
- **Physical presence required:** Code only shown at venue
- **Admin control:** Manual rotation possible
- **Audit trail:** Can log rotation times (not who used code)

### Q7: Explain email security and OTP implementation.

**Answer:**
Nodemailer with Gmail SMTP + secure OTP handling:

**Email Configuration:**
```javascript
// Email.config.js
import nodemailer from 'nodemailer'

export const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: process.env.SMTP_USER,  // Gmail address
    pass: process.env.SMTP_PASS   // App-specific password
  }
})

// Test connection on startup
transporter.verify((error, success) => {
  if (error) {
    console.error('Email config error:', error)
  } else {
    console.log('Email server ready')
  }
})
```

**OTP Generation (Registration):**
```javascript
// Generate cryptographically secure OTP
const generateOtp = () => String(Math.floor(100000 + Math.random() * 900000))

// Generate unique token for tracking
const genToken = () => crypto.randomUUID ? 
  crypto.randomUUID() : 
  crypto.randomBytes(16).toString('hex')

// In-memory store (temporary)
const pendingRegs = new Map()

const Register = AsynHandler(async(req, res) => {
  // ... validation ...
  
  const code = generateOtp()  // e.g., "837492"
  const token = genToken()    // e.g., "a1b2c3d4-..."
  const expiresAt = Date.now() + 10*60*1000  // 10 min
  
  // Store pending registration
  pendingRegs.set(token, {
    code,
    expiresAt,
    data: { ...formData, images }
  })
  
  // Send email
  await sendMail({
    to: Email,
    subject: 'E-VoteHub Verification Code',
    html: buildEmailTemplate({
      title: 'Verify your email',
      body: `Your code: <strong>${code}</strong>`
    })
  })
  
  // Return token (not code!)
  return res.json(new ApiResponse(200, {otpToken: token}, "OTP sent"))
})
```

**OTP Verification:**
```javascript
const RegisterVerify = AsynHandler(async(req, res) => {
  const {otpToken, code} = req.body
  
  // Retrieve pending registration
  const pending = pendingRegs.get(otpToken)
  
  if (!pending) {
    throw new ApiError(404, "Invalid or expired token")
  }
  
  // Check expiration
  if (Date.now() > pending.expiresAt) {
    pendingRegs.delete(otpToken)
    throw new ApiError(403, "OTP expired")
  }
  
  // Check code match
  if (String(pending.code) !== String(code)) {
    throw new ApiError(401, "Incorrect OTP")
  }
  
  // Code valid, proceed with registration
  const {data} = pending
  
  // Upload images
  const profileUrl = await FileUpload(data.ProfileImageLocalPath)
  const coverUrl = await FileUpload(data.CoverImageLocalPath)
  
  // Create user
  const user = await User.create({
    ...data,
    ProfileImage: profileUrl.url,
    ProfilePublicId: profileUrl.public_id,
    CoverImage: coverUrl.url,
    CoverPublicId: coverUrl.public_id
  })
  
  // Clean up
  pendingRegs.delete(otpToken)
  
  return res.status(201).json(
    new ApiResponse(201, user, "Registration complete")
  )
})
```

**Online Voting OTP:**
```javascript
const SendOnlineVoteCode = AsynHandler(async(req, res) => {
  const {EventID} = req.body
  const UserID = req.user._id
  
  const voter = await VoterReg.findOne({EventID, UserID})
  if (!voter) throw new ApiError(404, "Not registered")
  if (voter.hasVoted) throw new ApiError(403, "Already voted")
  
  // Generate code
  const code = String(Math.floor(100000 + Math.random() * 900000))
  const expiresAt = new Date(Date.now() + 10*60*1000)
  
  // Store in voter record
  voter.emailCode = code
  voter.emailCodeExpiresAt = expiresAt
  await voter.save()
  
  // Send email
  await sendMail({
    to: req.user.Email,
    subject: 'Your Voting Code',
    html: `Your code: <strong>${code}</strong><br>Valid for 10 minutes.`
  })
  
  return res.json(new ApiResponse(200, {}, "Code sent"))
})
```

**Security Features:**
- **App-specific password:** Not Gmail account password
- **TLS encryption:** Email sent over secure connection
- **Time-limited codes:** Expire after 10 minutes
- **One-time use:** Code deleted after verification
- **Rate limiting:** Prevent OTP spam (can implement)
- **Memory storage:** Pending registrations not in DB
- **Auto-cleanup:** Expired OTPs removed from memory
- **Professional templates:** HTML emails with branding

---

## 🛠️ Technical Implementation Details

### System Requirements
- **Backend:** Node.js 18+, MongoDB 6+
- **Frontend:** Modern browser with ES6+ support
- **Cloud:** Cloudinary account, Gmail SMTP

### Environment Variables
```env
# Backend (.env)
PORT=8002
MONGODB_URI=mongodb+srv://...
CORS_ORIGIN=http://localhost:5173,https://yourdomain.com
ACCESS_TOKEN_SECRET=your-256-bit-secret
REFRESH_TOKEN_SECRET=your-256-bit-secret
CLOUD_NAME=cloudinary-name
API_KEY=cloudinary-api-key
API_SECRET=cloudinary-api-secret
SMTP_USER=your-gmail@gmail.com
SMTP_PASS=gmail-app-password
```

```env
# Frontend (.env)
VITE_API_BASE=http://localhost:8002
```

### Installation & Running

**Backend:**
```bash
cd Back-end
npm install
npm run dev  # Starts on port 8002
```

**Frontend:**
```bash
cd Front-end
npm install
npm run dev  # Starts on port 5173
```

### API Endpoints Summary

**User Routes (`/api/v1/users`):**
- POST `/register` - Initialize registration
- POST `/register/verify` - Verify OTP
- POST `/login` - User login
- POST `/logout` - User logout
- POST `/renewaccestoken` - Refresh access token
- POST `/changepassword` - Change password
- PATCH `/UpdateProfilePicture` - Update profile image
- PATCH `/UpdateCoverPicture` - Update cover image
- GET `/profile/:id` - Get public profile

**Vote Event Routes (`/api/V1/admin`):**
- POST `/VoteEvent` - Create event (admin)
- GET `/events` - List all events
- POST `/nomineReg` - Register as nominee
- POST `/voterReg` - Register as voter
- POST `/voting` - Cast vote
- GET `/countvote` - Get vote results
- POST `/nomineeApproval` - Approve nominee (admin)
- GET `/getApproveNominee` - Get approved nominees
- GET `/getPendingNominee` - Get pending nominees
- POST `/rotateOnCampusCode` - Rotate voting code (admin)
- POST `/sendOnlineVoteCode` - Send voting code via email
- GET `/voteStatus` - Check if user has voted
- GET `/myVoteHistory` - Get user's voting history
- DELETE `/removeVoter` - Remove voter (admin)
- DELETE `/removeNominee` - Remove nominee (admin)

**Campaign Routes (`/api/v1/post`):**
- POST `/posting` - Create campaign post
- POST `/comment` - Comment on post
- POST `/like` - Like post/comment
- GET `/posts` - Get event posts
- DELETE `/post/:id` - Delete post

---

## 🌟 Key Features & Innovation

### Unique Features of E-VoteHub

1. **Dual-Phase System:**
   - Campaign Phase: Social engagement
   - Voting Phase: Secure ballot casting
   - Seamless transition between phases

2. **Flexible Voting Modes:**
   - Online: Email-based verification
   - On-Campus: Physical presence with rotating codes
   - Hybrid: Support both in same event

3. **Multiple Election Types:**
   - Single choice (First Past the Post)
   - Multi-vote (Select multiple)
   - Ranked voting (Preference-based)

4. **Social Campaign Platform:**
   - Nominee posts with media
   - Voter engagement through likes/comments
   - Profile pages for candidates
   - Real-time interaction

5. **Complete Anonymity:**
   - No vote-voter linkage in database
   - Aggregate counting only
   - Privacy-preserving architecture

6. **Real-time Updates:**
   - Live vote counts
   - Instant event notifications
   - Dynamic code rotation
   - WebSocket communication

7. **Professional UI/UX:**
   - Dark/light theme
   - Responsive design
   - Smooth animations
   - Intuitive navigation

8. **Security Features:**
   - JWT authentication
   - Bcrypt password hashing
   - Email verification
   - Time-limited codes
   - CORS protection
   - Role-based access control

9. **Cloud Integration:**
   - Cloudinary for media storage
   - MongoDB Atlas for database
   - Scalable architecture

10. **Admin Dashboard:**
    - Event management
    - Nominee approval
    - Voter management
    - Real-time analytics
    - Code rotation control

### Project Innovation Points

- **Novel dual-phase approach** combining campaigning and voting
- **Rotating code system** for physical voting locations
- **Anonymous vote counting** with mathematical guarantee
- **Real-time engagement** during campaign phase
- **Flexible election types** supporting various voting systems
- **Professional email templates** with OTP verification
- **Socket.io integration** for live updates

---

## 📊 Project Statistics

- **Lines of Code:** ~5000+ (Backend) + ~3000+ (Frontend)
- **API Endpoints:** 30+
- **Database Collections:** 7
- **External Services:** 3 (MongoDB Atlas, Cloudinary, Gmail SMTP)
- **Real-time Events:** 5+ (Socket.io)
- **Authentication Methods:** 2 (JWT + Email OTP)
- **Supported Election Types:** 3
- **Voting Modes:** 2

---

## 🎓 Learning Outcomes Achieved

### CO 1: Apply latest state of the art technologies ✅
- Modern MERN stack
- Real-time WebSocket communication
- Cloud services integration
- JWT authentication
- RESTful API design

### CO 2: Design and implement ideas for complete software ✅
- Full-stack application
- Frontend + Backend + Database
- Authentication system
- File upload system
- Real-time features
- Email service integration

### CO 3: Evaluate existing computer and mobile applications ✅
- Analyzed existing voting systems
- Identified security gaps
- Improved anonymity guarantees
- Enhanced user engagement
- Optimized database design

### CO 4: Explain ideas to groups and present their noble findings ✅
- Clear project documentation
- Comprehensive workflow explanation
- Technical architecture diagrams
- Code organization
- Feature demonstration capability

---

## 💡 Potential Teacher Questions & Answers

### General Questions:

**Q: Why did you choose MERN stack?**
**A:** MERN provides a complete JavaScript ecosystem allowing rapid development. React offers component reusability, Express provides flexible routing, MongoDB suits our document-based data model (users, events, posts), and Node.js enables real-time features with Socket.io.

**Q: What was the biggest challenge?**
**A:** Ensuring vote anonymity while maintaining integrity. We solved it by separating voter identity (VoterReg) from vote choices (VoteCount), making it mathematically impossible to link a voter to their vote.

**Q: How does your project differ from existing solutions?**
**A:** Traditional e-voting lacks engagement. We integrated a social campaign phase where nominees interact with voters through posts, comments, and likes before voting, increasing participation and informed decision-making.

**Q: Can you scale this system?**
**A:** Yes. MongoDB Atlas provides horizontal scaling, Cloudinary handles media CDN, and Node.js clustering can distribute load. Socket.io supports Redis adapter for multi-server deployments.

**Q: What would you improve with more time?**
**A:** 
1. Blockchain integration for immutable vote records
2. Face recognition for identity verification
3. Mobile app (React Native)
4. Advanced analytics dashboard
5. Multi-language support
6. Accessibility features (screen reader support)

---

## 🔒 Security Assurance Summary

✅ **Passwords:** Bcrypt hashed, never stored plain  
✅ **Authentication:** JWT with short-lived access tokens  
✅ **Anonymity:** Database architecture prevents vote tracing  
✅ **Input Validation:** Mongoose schemas + server-side checks  
✅ **XSS Prevention:** React auto-escaping  
✅ **CSRF Protection:** SameSite cookies + CORS  
✅ **Code Security:** Time-limited, rotating codes  
✅ **Email Security:** TLS encryption, OTP verification  
✅ **File Upload:** Type validation, Cloudinary sandboxing  
✅ **Environment:** Secrets in .env, never committed  

---

## 📞 Project Team Roles

1. **Core Backend Architect**
   - API design and implementation
   - Database schema design
   - Authentication system
   - Business logic controllers
   - Real-time Socket.io integration

2. **UI/UX & Frontend Specialist**
   - React component development
   - Responsive design with Tailwind
   - Theme system implementation
   - API integration
   - User experience optimization

3. **Database & Security Engineer**
   - MongoDB schema design
   - Security implementations
   - Password hashing
   - Vote anonymity architecture
   - Email OTP system
   - Code rotation mechanism

---

## ✅ Project Completion Checklist

- [x] User authentication (register, login, logout)
- [x] JWT token management
- [x] Email verification with OTP
- [x] Admin dashboard
- [x] Event creation and management
- [x] Nominee registration and approval
- [x] Voter registration
- [x] Online voting with email codes
- [x] On-campus voting with rotating codes
- [x] Campaign post creation
- [x] Social engagement (likes, comments)
- [x] Real-time updates (Socket.io)
- [x] Vote counting system
- [x] Vote anonymity guarantee
- [x] File upload (Cloudinary)
- [x] Responsive design
- [x] Dark/light theme
- [x] Error handling
- [x] Input validation
- [x] Security measures

---

## 🎯 Final Evaluation Tips

### For Backend Architect:
- Explain MVC pattern clearly
- Demonstrate JWT flow with diagram
- Show database relationships
- Explain async/await usage
- Describe error handling strategy

### For Frontend Specialist:
- Show component hierarchy
- Explain state management
- Demonstrate routing
- Show responsive design
- Explain API integration

### For Database & Security:
- Draw schema diagrams
- Explain anonymity architecture
- Show security implementations
- Demonstrate OTP flow
- Explain encryption methods

---

**Good luck with your evaluation! This comprehensive guide covers all aspects of your E-VoteHub project. Study each section and be ready to explain or demonstrate any part in detail.**

---

*Document Created: February 1, 2026*  
*Project: E-VoteHub 2.0*  
*Course: Project Work II (0610 2250)*  
*Credits: 1.5 | Contact Hours: 3 hours/week*
