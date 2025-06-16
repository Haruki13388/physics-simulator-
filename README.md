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

export default App;
npm start
