# React Router

## What React Router solves

React Router maps browser URLs to React components without requiring a full page reload for normal client-side navigation.

    /              -> Home
    /login         -> Login
    /users/42      -> UserProfile
    /settings      -> Settings
    /admin         -> AdminDashboard

Mental model:

    navigation -> URL changes -> route matches -> component tree renders

Install:

    npm install react-router-dom

## BrowserRouter vs HashRouter

BrowserRouter:

    <BrowserRouter>
      <App />
    </BrowserRouter>

URL:

    https://example.com/users/42

It uses the browser History API. If the user directly visits /users/42, the server must be configured for SPA fallback so it returns index.html; React Router then handles the route. Otherwise the server can return 404.

HashRouter:

    <HashRouter>
      <App />
    </HashRouter>

URL:

    https://example.com/#/users/42

Everything after # is handled by the browser, so the server generally only needs to serve the base page.

| | BrowserRouter | HashRouter |
|---|---|---|
| URL | /users/42 | /#/users/42 |
| History API | Yes | No |
| Clean URLs | Yes | No |
| Server SPA fallback | Required | Usually not |
| Typical modern SPA choice | Yes | Less common |

Practical rule: use BrowserRouter for a normal production SPA. HashRouter is useful when the deployment environment cannot support SPA fallback.

## Basic routing

    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </BrowserRouter>

## Link navigation

For internal navigation, prefer React Router's Link over a normal anchor so navigation stays client-side.

    <Link to="/">Home</Link>
    <Link to="/settings">Settings</Link>

## useNavigate

Use useNavigate when application logic needs to navigate programmatically.

    const navigate = useNavigate();

    navigate("/");                 // navigate
    navigate(-1);                  // back
    navigate(1);                   // forward
    navigate("/", { replace: true });

replace is useful after login so the user does not simply navigate back to the login page.

## useParams

Dynamic route:

    <Route path="/users/:userId" element={<UserProfile />} />

For /users/42:

    const { userId } = useParams();

userId is "42". Route parameters are strings, so convert when necessary:

    const id = Number(userId);

Real API pattern:

    function UserProfile() {
      const { userId } = useParams();
      const [user, setUser] = useState(null);

      useEffect(() => {
        fetch(`/api/users/${userId}`)
          .then(res => res.json())
          .then(setUser);
      }, [userId]);

      if (!user) return <p>Loading...</p>;

      return (
        <div>
          <h1>{user.name}</h1>
          <p>{user.email}</p>
        </div>
      );
    }

## useLocation

useLocation gives information about the current URL.

Conceptually:

    {
      pathname: "/users/42",
      search: "?tab=posts",
      hash: "#comments",
      state: ...
    }

A common use case is reacting to route changes:

    const location = useLocation();

    useEffect(() => {
      trackPageView(location.pathname);
    }, [location.pathname]);

For query parameters, use useSearchParams:

    const [searchParams] = useSearchParams();
    const tab = searchParams.get("tab");

Do not confuse:

    /users/:userId       -> path parameter
    /users?userId=42     -> query parameter

## Protected routes

Mental model:

    /dashboard
        |
    ProtectedRoute
        |
    authenticated?
       /       \
     yes       no
      |         |
 Dashboard    /login

Basic implementation:

    function ProtectedRoute({ children }) {
      const { user } = useAuth();

      if (!user) {
        return <Navigate to="/login" replace />;
      }

      return children;
    }

Usage:

    <Route
      path="/dashboard"
      element={
        <ProtectedRoute>
          <Dashboard />
        </ProtectedRoute>
      }
    />

Navigate is the declarative redirect component.

## Complete authentication pattern

An auth context can expose the current user and login/logout operations:

    const AuthContext = createContext(null);

    function AuthProvider({ children }) {
      const [user, setUser] = useState(null);

      async function login(email, password) {
        const response = await fetch("/api/login", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ email, password })
        });

        if (!response.ok) {
          throw new Error("Login failed");
        }

        const user = await response.json();
        setUser(user);
      }

      function logout() {
        setUser(null);
      }

      return (
        <AuthContext.Provider value={{ user, login, logout }}>
          {children}
        </AuthContext.Provider>
      );
    }

    function useAuth() {
      return useContext(AuthContext);
    }

Router:

    function App() {
      return (
        <BrowserRouter>
          <Routes>
            <Route path="/login" element={<Login />} />

            <Route
              path="/"
              element={
                <ProtectedRoute>
                  <Home />
                </ProtectedRoute>
              }
            />

            <Route
              path="/users/:userId"
              element={
                <ProtectedRoute>
                  <UserProfile />
                </ProtectedRoute>
              }
            />

            <Route
              path="/settings"
              element={
                <ProtectedRoute>
                  <Settings />
                </ProtectedRoute>
              }
            />
          </Routes>
        </BrowserRouter>
      );
    }

## Preserve the original destination

If a logged-out user visits /settings, remember that location during the redirect:

    function ProtectedRoute({ children }) {
      const { user } = useAuth();
      const location = useLocation();

      if (!user) {
        return (
          <Navigate
            to="/login"
            replace
            state={{ from: location }}
          />
        );
      }

      return children;
    }

After login:

    function Login() {
      const { login } = useAuth();
      const location = useLocation();
      const navigate = useNavigate();

      async function handleLogin() {
        await login();

        const destination =
          location.state?.from?.pathname || "/";

        navigate(destination, { replace: true });
      }

      return <button onClick={handleLogin}>Login</button>;
    }

Flow:

    /settings
        -> not authenticated
        -> /login
        -> state.from = "/settings"
        -> login succeeds
        -> navigate("/settings")

## Authentication vs authorization

Authentication asks: Who are you?

Authorization asks: What are you allowed to access?

Example:

    USER  -> /dashboard
    ADMIN -> /admin

A protected route can check a role:

    function ProtectedRoute({ children, requiredRole }) {
      const { user } = useAuth();

      if (!user) {
        return <Navigate to="/login" replace />;
      }

      if (requiredRole && user.role !== requiredRole) {
        return <Navigate to="/forbidden" replace />;
      }

      return children;
    }

Usage:

    <Route
      path="/admin"
      element={
        <ProtectedRoute requiredRole="ADMIN">
          <AdminDashboard />
        </ProtectedRoute>
      }
    />

Critical: frontend route protection is not real security. A user can bypass React and call the API directly. The backend must independently enforce authorization.

## Nested routes and base route UI

Nested routes let a parent route provide shared UI while the matched child renders inside it.

Example:

    /dashboard
    /dashboard/profile
    /dashboard/settings

Routes:

    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>

        <Route index element={<DashboardHome />} />

        <Route path="profile" element={<Profile />} />

        <Route path="settings" element={<Settings />} />

      </Route>
    </Routes>

Child paths are relative. React Router combines the parent and child:

    /dashboard + profile
          -> /dashboard/profile

The parent layout uses Outlet:

    function DashboardLayout() {
      return (
        <div>
          <Navbar />

          <div className="layout">
            <Sidebar />

            <main>
              <Outlet />
            </main>
          </div>
        </div>
      );
    }

Outlet means: render the matched child route here.

So:

    /dashboard
        -> DashboardLayout
        -> Outlet
        -> DashboardHome

    /dashboard/settings
        -> DashboardLayout
        -> Outlet
        -> Settings

### Index route

    <Route index element={<DashboardHome />} />

The index route is the default child route.

    /dashboard
       |- index    -> DashboardHome
       |- profile  -> Profile
       '- settings -> Settings

### Root application layout

The same pattern can provide application-wide UI:

    <BrowserRouter>
      <Routes>
        <Route element={<AppLayout />}>
          <Route path="/" element={<Home />} />
          <Route path="/login" element={<Login />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Route>
      </Routes>
    </BrowserRouter>

    function AppLayout() {
      return (
        <>
          <Navbar />
          <Outlet />
          <Footer />
        </>
      );
    }

Mental model:

    BrowserRouter
         |
      AppLayout
       |- Navbar
       |- Outlet
       |    |- Home
       |    |- Login
       |    '- Dashboard
       '- Footer

## SDE-2 cheat sheet

    useNavigate()
        -> "I want to CHANGE the URL"

    useParams()
        -> "I want a VALUE from the URL path"

    useLocation()
        -> "I want INFORMATION about the CURRENT URL"

    <Navigate />
        -> "REDIRECT this user"

    <Outlet />
        -> "RENDER the matched CHILD route here"

    index route
        -> "DEFAULT child route"

## Best practices

- Prefer BrowserRouter for normal modern SPAs.
- Use Link/NavLink for user-driven internal navigation.
- Use useNavigate for programmatic navigation.
- Use path parameters for resource identity such as /users/:userId.
- Use query parameters for filters, sorting, pagination and tabs.
- Use a reusable ProtectedRoute for authentication checks.
- Preserve the original destination when redirecting to login when useful.
- Keep authentication state centralized.
- Enforce authorization on the backend; frontend protection is not security.
- Use nested routes + Outlet for shared/base route UI.
- Use index routes for a parent's default page.
- Configure SPA fallback correctly when deploying BrowserRouter.
