# physics-simulator-
// src/PhysicsSimulator.js
import React from "react";
import { Canvas } from "@react-three/fiber";
import { Physics, useBox, usePlane } from "@react-three/cannon";

function Box(props) {
  const [ref] = useBox(() => ({ mass: 1, position: [0, 5, 0], ...props }));
  return (
    <mesh ref={ref} castShadow>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="orange" />
    </mesh>
  );
}

function Plane(props) {
  const [ref] = usePlane(() => ({ ...props }));
  return (
    <mesh ref={ref} receiveShadow>
      <planeGeometry args={[10, 10]} />
      <meshStandardMaterial color="lightblue" />
    </mesh>
  );
}

export default function PhysicsSimulator() {
  return (
    <Canvas shadows camera={{ position: [0, 5, 10], fov: 60 }}>
      <ambientLight />
      <directionalLight position={[0, 10, 5]} intensity={1} castShadow />
      <Physics>
        <Box />
        <Plane rotation={[-Math.PI / 2, 0, 0]} position={[0, -0.5, 0]} />
      </Physics>
    </Canvas>
  );
}
// src/App.js
import React from "react";
import PhysicsSimulator from "./PhysicsSimulator";

function App() {
  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      <PhysicsSimulator />
    </div>
  );
}

export default App;
import React from "react";
import PhysicsSimulator from "./PhysicsSimulator";

function App() {
  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      <PhysicsSimulator />
    </div>
  );
}
import React, { useRef, useEffect, useContext } from 'react';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { AppContext } from './context/AppContext';

export default function UniverseCanvas() {
  const mountRef = useRef(null);
  const { selectedObject, setSelectedObject, physicsSettings } = useContext(AppContext);

  useEffect(() => {
    // ... (Three.js scene setup as before)
    // Add raycaster for object selection

    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    const onPointerDown = (event) => {
      // Convert pointer event to normalized device coordinates
      const rect = renderer.domElement.getBoundingClientRect();
      mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
      mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);

      const intersects = raycaster.intersectObjects(planets.concat([star]), true);
      if (intersects.length > 0) {
        setSelectedObject(intersects[0].object.userData);
      }
    };

    renderer.domElement.addEventListener('pointerdown', onPointerDown);

    // ... (animation loop + cleanup)
    return () => {
      renderer.domElement.removeEventListener('pointerdown', onPointerDown);
      // ... (cleanup)
    };
  }, [physicsSettings, setSelectedObject]);

  return (
    <div ref={mountRef} style={{ width: "100%", height: "600px" }} />
  );
}import React, { useContext } from 'react';
import { AppContext } from './context/AppContext';

export default function ObjectInfoPanel() {
  const { selectedObject, enterPlanet } = useContext(AppContext);

  if (!selectedObject) return null;

  return (
    <div className="object-info-panel">
      <h2>{selectedObject.name}</h2>
      <ul>
        <li>Type: {selectedObject.type}</li>
        <li>Mass: {selectedObject.mass} kg</li>
        <li>Radius: {selectedObject.radius} km</li>
        <li>Orbital Period: {selectedObject.orbitalPeriod} days</li>
        {/* Add more stats as desired */}
      </ul>
      <p>{selectedObject.description}</p>
      <button onClick={() => enterPlanet(selectedObject)}>Enter Planet</button>
    </div>
  );
}import React, { useState, useContext } from 'react';
import { AppContext } from './context/AppContext';

/**
 * EquationGenerator allows the user to input a physics formula
 * and applies it to the simulation via context.
 */
export default function EquationGenerator() {
  const [equation, setEquation] = useState('');
  const { applyEquation } = useContext(AppContext);

  const handleInput = (e) => setEquation(e.target.value);

  const handleApply = () => {
    if (equation.trim()) {
      applyEquation(equation);
      setEquation('');
    }
  };

  return (
    <div className="equation-generator">
      <h4>Equation Generator</h4>
      <input
        type="text"
        value={equation}
        onChange={handleInput}
        placeholder="Type or paste a physics equation"
        spellCheck={false}
      />
      <button onClick={handleApply} disabled={!equation.trim()}>
        Apply
      </button>
    </div>
  );
}import React, { useEffect, useState } from 'react';

/**
 * IntroAnimation shows a Big Bang animation as an intro splash,
 * then fades out after a few seconds.
 */
export default function IntroAnimation() {
  const [visible, setVisible] = useState(true);

  useEffect(() => {
    const timer = setTimeout(() => setVisible(false), 4000); // Show for 4 seconds
    return () => clearTimeout(timer);
  }, []);

  if (!visible) return null;

  return (
    <div className="intro-animation" style={{
      position: 'fixed', zIndex: 1000, top: 0, left: 0,
      width: '100vw', height: '100vh', background: 'black', display: 'flex',
      flexDirection: 'column', alignItems: 'center', justifyContent: 'center'
    }}>
      <img
        src="/bigbang.gif"
        alt="Big Bang Animation"
        style={{ maxWidth: '80vw', maxHeight: '60vh', borderRadius: '24px' }}
      />
      <h1 style={{
        color: 'white', marginTop: '2rem', letterSpacing: '0.1em',
        fontWeight: 700, fontSize: '2.5rem', textShadow: '0 0 20px #00f, 0 0 40px #fff'
      }}>
        The Universe Begins...
      </h1>
    </div>
  );
}import React, { createContext, useState } from 'react';

/**
 * AppContext provides global simulation state and actions.
 */
export const AppContext = createContext();

export function AppProvider({ children }) {
  // Currently selected celestial object (planet, star, etc.)
  const [selectedObject, setSelectedObject] = useState(null);

  // Basic physics settings
  const [physicsSettings, setPhysicsSettings] = useState({
    gravity: 9.81,
    speed: 1.0,
  });

  // Advanced physics settings
  const [advancedSettings, setAdvancedSettings] = useState({
    massModifier: 1.0,
    timeDilation: 1.0,
  });

  // Example placeholder for searching celestial objects
  const searchObjects = (query) => {
    // TODO: Implement real search logic and setSelectedObject(matched)import React from 'react';
import { render, screen } from '@testing-library/react';
import UniverseCanvas from './UniverseCanvas';

describe('UniverseCanvas', () => {
  test('renders the canvas container', () => {
    render(<UniverseCanvas />);
    // The main div should be in the document
    const canvasDiv = screen.getByRole('region', { hidden: true }) || document.querySelector('div');
    expect(canvasDiv).toBeInTheDocument();
  });

  test('mounts Three.js renderer', () => {
    const { container } = render(<UniverseCanvas />);
    // Look for a canvas element (WebGLRenderer)
    const canvas = container.querySelector('canvas');
    expect(canvas).toBeInTheDocument();
  });

  // Optionally, test resizing or unmount cleanup here
});import React from 'react';
import { AppProvider } from './context/AppContext';
import UniverseCanvas from './UniverseCanvas';
import Toolbar from './Toolbar';
import SearchBar from './SearchBar';
import ObjectInfoPanel from './ObjectInfoPanel';
import EquationGenerator from './EquationGenerator';
import IntroAnimation from './IntroAnimation';

export default function App() {
  return (
    <AppProvider>
      <IntroAnimation />
      <div style={{ padding: '16px', background: '#111', minHeight: '100vh' }}>
        <h1 style={{ color: '#fff', textAlign: 'center' }}>Physics Simulator</h1>
        <SearchBar />
        <Toolbar />
        <div style={{ margin: '32px 0' }}>
          <UniverseCanvas />
        </div>
        <ObjectInfoPanel />
        <EquationGenerator />
      </div>
    </AppProvider>
  );
}



