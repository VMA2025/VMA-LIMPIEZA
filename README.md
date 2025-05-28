// limpieza-web-app: App web de limpieza diaria con roles, reportes y asignación de tareas

// 📁 src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);

// 📁 src/App.jsx
import { useState, useEffect } from 'react';
import { auth, db } from './firebase';
import { onAuthStateChanged, signOut } from 'firebase/auth';
import Auth from './components/Auth';
import TaskList from './components/TaskList';
import Reporte from './components/Reporte';
import AdminPanel from './components/AdminPanel';
import { doc, getDoc } from 'firebase/firestore';

function App() {
  const [user, setUser] = useState(null);
  const [role, setRole] = useState('');
  const [view, setView] = useState('main');

  useEffect(() => {
    const unsub = onAuthStateChanged(auth, async (user) => {
      setUser(user);
      if (user) {
        const userDoc = await getDoc(doc(db, 'usuarios', user.uid));
        if (userDoc.exists()) {
          setRole(userDoc.data().rol);
        }
      }
    });
    return () => unsub();
  }, []);

  const renderContent = () => {
    if (role === 'supervisor') {
      if (view === 'admin') return <AdminPanel />;
      if (view === 'reporte') return <Reporte />;
      return (
        <div>
          <div className="flex gap-4 mb-4">
            <button className="bg-gray-200 px-4 py-2" onClick={() => setView('reporte')}>Ver Reportes</button>
            <button className="bg-gray-200 px-4 py-2" onClick={() => setView('admin')}>Panel Admin</button>
          </div>
          <Reporte />
        </div>
      );
    }
    return <TaskList user={user} />;
  };

  return (
    <div className="p-4">
      {user ? (
        <div>
          <div className="flex justify-between items-center">
            <h1 className="text-xl font-bold">Bienvenido, {user.email}</h1>
            <button onClick={() => signOut(auth)} className="text-red-500">Cerrar sesión</button>
          </div>
          {renderContent()}
        </div>
      ) : (
        <Auth />
      )}
    </div>
  );
}

export default App;

// 📁 src/components/AdminPanel.jsx
import { useEffect, useState } from 'react';
import { db } from '../firebase';
import { collection, getDocs, doc, updateDoc, addDoc } from 'firebase/firestore';

function AdminPanel() {
  const [usuarios, setUsuarios] = useState([]);
  const [descripcion, setDescripcion] = useState('');
  const [zona, setZona] = useState('');
  const [fecha, setFecha] = useState('');
  const [asignadoA, setAsignadoA] = useState('');

  useEffect(() => {
    const fetchUsers = async () => {
      const querySnapshot = await getDocs(collection(db, 'usuarios'));
      const usersData = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setUsuarios(usersData);
    };
    fetchUsers();
  }, []);

  const cambiarRol = async (id, nuevoRol) => {
    await updateDoc(doc(db, 'usuarios', id), { rol: nuevoRol });
    setUsuarios(usuarios.map(u => u.id === id ? { ...u, rol: nuevoRol } : u));
  };

  const crearTarea = async () => {
    if (!descripcion || !zona || !fecha || !asignadoA) return;
    await addDoc(collection(db, 'tareas'), {
      descripcion,
      zona,
      fecha,
      asignadoA,
      completada: false,
      evidencia: ''
    });
    setDescripcion('');
    setZona('');
    setFecha('');
    setAsignadoA('');
    alert('Tarea creada y asignada');
  };

  return (
    <div className="mt-4 space-y-8">
      <div>
        <h2 className="text-lg font-bold mb-2">Usuarios</h2>
        {usuarios.map(user => (
          <div key={user.id} className="border p-2 flex justify-between">
            <span>{user.email} ({user.rol})</span>
            <button onClick={() => cambiarRol(user.id, user.rol === 'limpiador' ? 'supervisor' : 'limpiador')} className="text-blue-500">
              Cambiar a {user.rol === 'limpiador' ? 'supervisor' : 'limpiador'}
            </button>
          </div>
        ))}
      </div>

      <div>
        <h2 className="text-lg font-bold mb-2">Crear y asignar tarea</h2>
        <input value={descripcion} onChange={e => setDescripcion(e.target.value)} placeholder="Descripción" className="border p-2 mr-2" />
        <input value={zona} onChange={e => setZona(e.target.value)} placeholder="Zona" className="border p-2 mr-2" />
        <input value={fecha} onChange={e => setFecha(e.target.value)} placeholder="Fecha" className="border p-2 mr-2" />
        <select value={asignadoA} onChange={e => setAsignadoA(e.target.value)} className="border p-2 mr-2">
          <option value="">Asignar a...</option>
          {usuarios.map(user => (
            <option key={user.id} value={user.id}>{user.email}</option>
          ))}
        </select>
        <button onClick={crearTarea} className="bg-green-500 text-white px-4 py-2">Crear</button>
      </div>
    </div>
  );
}

export default AdminPanel;

// 📁 src/components/Auth.jsx
// (sin cambios)

// 📁 src/components/Reporte.jsx
// (sin cambios)
