# 🔑 Convertir Usuario Existente en Admin

## Opción 1: Desde Firebase Console (Más Fácil)

1. Ve a [Firebase Console](https://console.firebase.google.com/)
2. Proyecto: **datos-de-jose-para-silo**
3. Ve a **Authentication** → **Users**
4. Busca tu usuario y **copia el UID** (el código largo)
5. Ve a **Firestore Database** → **Data**
6. Busca la colección `usersMetadata`
7. Si tu usuario NO tiene documento ahí:
   - Clic en **Add document**
   - Document ID: [pega el UID]
   - Campos:
     ```
     email: "tu@email.com" (string)
     displayName: "Tu Nombre" (string)
     role: "admin" (string)
     isActive: true (boolean)
     createdAt: [timestamp]
     createdBy: "system" (string)
     ```
8. Si tu usuario YA tiene documento:
   - Haz clic en el documento
   - Edita el campo `role` y cámbialo a `"admin"`
9. **Recarga la página** en tu app y verás el botón "Gestión Usuarios"

---

## Opción 2: Desde la Consola del Navegador (Más Rápido)

1. **Inicia sesión** con el usuario que quieres hacer admin
2. Abre la **consola del navegador** (F12)
3. Ve a la pestaña **Console**
4. Pega y ejecuta este código:

```javascript
(async () => {
  const { getFirestore, doc, setDoc, getDoc, Timestamp } = await import("https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js");
  const { getAuth } = await import("https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js");
  
  const db = getFirestore();
  const auth = getAuth();
  const uid = auth.currentUser.uid;
  
  // Verificar si ya existe metadata
  const userDoc = await getDoc(doc(db, 'usersMetadata', uid));
  
  if (userDoc.exists()) {
    // Solo actualizar el rol
    await setDoc(doc(db, 'usersMetadata', uid), {
      ...userDoc.data(),
      role: "admin"
    });
    console.log("✅ Rol actualizado a admin");
  } else {
    // Crear metadata completa
    await setDoc(doc(db, 'usersMetadata', uid), {
      email: auth.currentUser.email,
      displayName: auth.currentUser.displayName || "Admin",
      role: "admin",
      isActive: true,
      createdAt: Timestamp.now(),
      createdBy: "system"
    });
    console.log("✅ Usuario convertido a admin");
  }
  
  alert("¡Listo! Recarga la página (F5) para ver los cambios.");
})();
```

5. **Recarga la página** (F5)
6. Deberías ver el botón **"Gestión Usuarios"** en el header

---

## ✅ Verificación

Después de hacer admin a tu usuario:

- ✅ Debes ver el botón **"Gestión Usuarios"** en `app.html` y `desechos.html`
- ✅ Puedes hacer clic y acceder a la gestión de usuarios
- ✅ Puedes ver botones de eliminar/editar en las tablas
- ✅ Puedes seguir usando la app normalmente con todas las funciones

---

## 🎯 Resumen

El botón "Gestión Usuarios" **ya está en app.html**, solo que está oculto. Cuando conviertas tu usuario a admin, el botón aparecerá automáticamente y podrás:

1. Seguir usando `app.html` y `desechos.html` normalmente
2. Ver y usar el botón "Gestión Usuarios" para administrar otros usuarios
3. Eliminar y editar cualquier dato (ahora tienes permisos de admin)
