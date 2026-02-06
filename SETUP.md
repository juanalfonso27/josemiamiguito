# 🚀 Guía Rápida de Configuración - Admin CRM

## Paso 1: Desplegar Reglas de Firestore

1. Abre [Firebase Console](https://console.firebase.google.com/)
2. Proyecto: **datos-de-jose-para-silo**
3. Ve a **Firestore Database** → **Reglas**
4. Copia y pega el contenido de `firestore.rules`:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    function isAdmin() {
      return request.auth != null
        && exists(/databases/$(database)/documents/usersMetadata/$(request.auth.uid))
        && get(/databases/$(database)/documents/usersMetadata/$(request.auth.uid)).data.role == 'admin'
        && get(/databases/$(database)/documents/usersMetadata/$(request.auth.uid)).data.isActive == true;
    }
    
    function isActiveUser() {
      return request.auth != null
        && exists(/databases/$(database)/documents/usersMetadata/$(request.auth.uid))
        && get(/databases/$(database)/documents/usersMetadata/$(request.auth.uid)).data.isActive == true;
    }
    
    match /usersMetadata/{userId} {
      allow read: if request.auth != null && isActiveUser();
      allow create, update, delete: if isAdmin();
    }
    
    match /users/{userId}/{document=**} {
      allow read: if request.auth != null && isActiveUser();
      allow create: if request.auth != null && isActiveUser() && request.auth.uid == userId;
      allow update, delete: if isAdmin();
    }
  }
}
```

5. Haz clic en **Publicar**

---

## Paso 2: Crear Usuario Admin Inicial

### Opción A: Firebase Console (Recomendado)

1. **Authentication** → **Users** → **Add user**
   - Email: tu email
   - Password: tu contraseña
   - Copia el **UID** del usuario creado

2. **Firestore Database** → **Data** → **Start collection**
   - Collection ID: `usersMetadata`
   - Document ID: [pega el UID copiado]
   - Campos:
     ```
     email: "tu@email.com" (string)
     displayName: "Tu Nombre" (string)
     role: "admin" (string)
     isActive: true (boolean)
     createdAt: [timestamp - usa el botón de timestamp]
     createdBy: "system" (string)
     ```

3. **Save**

### Opción B: Script en Consola del Navegador

1. Inicia sesión en tu app con cualquier usuario existente
2. Abre la consola del navegador (F12)
3. Ve a la pestaña **Console**
4. Pega y ejecuta este código:

```javascript
// Ejecutar en la consola del navegador
(async () => {
  const { getFirestore, doc, setDoc, Timestamp } = await import("https://www.gstatic.com/firebasejs/11.1.0/firebase-firestore.js");
  const { getAuth } = await import("https://www.gstatic.com/firebasejs/11.1.0/firebase-auth.js");
  
  const db = getFirestore();
  const auth = getAuth();
  const uid = auth.currentUser.uid;
  
  await setDoc(doc(db, 'usersMetadata', uid), {
    email: auth.currentUser.email,
    displayName: "Admin Principal",
    role: "admin",
    isActive: true,
    createdAt: Timestamp.now(),
    createdBy: "system"
  });
  
  alert("✅ Usuario admin creado. Recarga la página para ver los cambios.");
})();
```

---

## Paso 3: Verificar Instalación

1. **Recarga la página** (F5)
2. Deberías ver el botón **"Gestión Usuarios"** en el header
3. Haz clic en **"Gestión Usuarios"**
4. Deberías ver la interfaz de administración de usuarios

---

## Paso 4: Crear Usuarios de Prueba

1. En **Gestión Usuarios**, clic en **"Registrar Usuario"**
2. Completa el formulario:
   - Email: usuario@test.com
   - Nombre: Usuario Prueba
   - Rol: Uploader
   - Contraseña: test123
3. Haz clic en **"Registrar Usuario"**

---

## Paso 5: Probar Permisos

1. **Cierra sesión** del admin
2. **Inicia sesión** con el usuario de prueba (usuario@test.com / test123)
3. Verifica que:
   - ❌ NO veas el botón "Gestión Usuarios"
   - ❌ NO veas botones de eliminar/editar en las tablas
   - ✅ Puedas crear nuevas cargas/desechos
   - ✅ Puedas ver los datos existentes

---

## ✅ Configuración Completa

Tu sistema CRM está listo para usar. Ahora puedes:

- ✅ Registrar usuarios desde la interfaz de admin
- ✅ Asignar roles (admin, uploader, approver)
- ✅ Controlar quién puede eliminar/modificar datos
- ✅ Activar/desactivar usuarios según sea necesario

---

## 🆘 Solución de Problemas

### "Permission denied" al intentar crear usuarios

→ Verifica que las reglas de Firestore estén publicadas correctamente

### No veo el botón "Gestión Usuarios"

→ Verifica que tu usuario tenga `role: "admin"` en la colección `usersMetadata`

### Error al crear usuario: "email-already-in-use"

→ El email ya está registrado. Usa otro email o elimina el usuario existente desde Firebase Console

### Usuario desactivado no puede iniciar sesión

→ Esto es correcto. Solo el admin puede reactivar usuarios desde "Gestión Usuarios"
