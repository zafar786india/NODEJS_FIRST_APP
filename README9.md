import { BrowserRouter as Router, Routes, Route, Link } from "react-router-dom";

import React from "react";
import { BrowserRouter as Router, Routes, Route, Link } from "react-router-dom";

const Home = () => <h2>Home Page</h2>;
const About = () => <h2>About Page</h2>;
const Contact = () => <h2>Contact Page</h2>;

const App = () => {
  return (
    <Router>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </Router>
  );
};

export default App;
const User = ({ params }) => <h2>User ID: {params.userId}</h2>;

<Routes>
  <Route path="/user/:userId" element={<User />} />
</Routes>;
const Dashboard = () => (
  <div>
    <h2>Dashboard</h2>
    <Routes>
      <Route path="profile" element={<h3>Profile</h3>} />
      <Route path="settings" element={<h3>Settings</h3>} />
    </Routes>
  </div>
);

<Routes>
  <Route path="/dashboard/*" element={<Dashboard />} />
</Routes>;
import { useNavigate } from "react-router-dom";

const NavigateButton = () => {
  const navigate = useNavigate();
  return <button onClick={() => navigate("/about")}>Go to About</button>;
};
<Route path="*" element={<h2>Page Not Found</h2>} />

