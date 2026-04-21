Como arquitecto de software, he diseñado una metodología integral para este proyecto. Vamos a estructurar el desarrollo de **bdcrudruleta0310** utilizando un enfoque de **Ingeniería de Software Basada en Agentes**.

Para la práctica guiada, utilizaremos **Antigravity** como el motor de orquestación de lógica. En este contexto, los Agentes no son solo clases; son entidades con responsabilidades únicas que se comunican para mantener el estado de la aplicación limpio.

---

## Metodología de Agentes: Roles y Skills

Antes de tocar el código, definimos la estructura operativa del sistema:

| Agente | Rol | Skill (Habilidad Principal) |
| :--- | :--- | :--- |
| **Agente Director** | Orquestador de UI | Maneja el flujo de navegación y la renderización de vistas. |
| **Agente Archivista** | Gestor de Datos | Traduce objetos Dart a documentos de Firestore y viceversa. |
| **Agente Conector** | Firebase Liaison | Gestiona la conexión inicial y la salud del túnel con la consola. |

---

## Fase 1: Configuración de Infraestructura

<Steps>
{/* Reason: El orden es crítico para evitar errores de compilación y conexión con Firebase. */}
  <Step title="Creación del Entorno" subtitle="Consola de comandos">
    Ejecuta en tu terminal:
    ```bash
    flutter create crudruleta
    cd crudruleta
    ```
  </Step>
  <Step title="Configuración en Firebase Console" subtitle="Consola Web">
    1. Ve a [Firebase Console](https://console.firebase.google.com/).
    2. Crea el proyecto **bdcrudruleta0310**.
    3. Habilita **Cloud Firestore** en modo de prueba.
    4. Registra tu app (Android/iOS) y descarga el archivo `google-services.json` en `android/app/`.
  </Step>
  <Step title="Integración de Librerías" subtitle="pubspec.yaml">
    Agrega las dependencias core. Abre `pubspec.yaml` y añade:
    ```yaml
    dependencies:
      flutter:
        sdk: flutter
      firebase_core: ^2.24.0
      cloud_firestore: ^4.14.0
      antigravity: ^1.0.0 # Motor de estados y agentes
    ```
    Luego ejecuta `flutter pub get`.
  </Step>
</Steps>

---

## Fase 2: Estructura de Carpetas (Arquitectura de Agentes)

Para que el proyecto sea escalable y cumpla con la metodología solicitada, organizaremos el código así:

```text
lib/
├── agents/             # Lógica de Antigravity (Roles y Skills)
│   ├── data_agent.dart
│   └── auth_agent.dart
├── models/             # Estructura de datos
│   └── user_model.dart
├── ui/                 # Interfaces de usuario
│   ├── user_form.dart
│   └── user_list.dart
└── main.dart           # Punto de entrada
```

---

## Fase 3: Implementación de Código Funcional

### 1. El Modelo de Datos (`models/user_model.dart`)
```dart
class UserModel {
  String? id;
  String nombre;
  String fechaNacimiento;
  String numeroTelefono;

  UserModel({this.id, required this.nombre, required this.fechaNacimiento, required this.numeroTelefono});

  // Skill del Archivista: Convertir de JSON a Objeto
  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "fecha_nacimiento": fechaNacimiento,
    "numero_telefono": numeroTelefono,
  };

  factory UserModel.fromMap(String id, Map<String, dynamic> map) => UserModel(
    id: id,
    nombre: map['nombre'] ?? '',
    fechaNacimiento: map['fecha_nacimiento'] ?? '',
    numeroTelefono: map['numero_telefono'] ?? '',
  );
}
```

### 2. El Agente Archivista (`agents/data_agent.dart`)
Este agente posee el **Skill de CRUD**.

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/user_model.dart';

class DataAgent {
  final FirebaseFirestore _db = FirebaseFirestore.instance;

  // CREATE
  Future<void> createUser(UserModel user) async {
    await _db.collection('usuarios').add(user.toMap());
  }

  // READ (Stream para tiempo real)
  Stream<List<UserModel>> getUsers() {
    return _db.collection('usuarios').snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => UserModel.fromMap(doc.id, doc.data())).toList());
  }

  // UPDATE
  Future<void> updateUser(UserModel user) async {
    await _db.collection('usuarios').doc(user.id).update(user.toMap());
  }

  // DELETE
  Future<void> deleteUser(String id) async {
    await _db.collection('usuarios').doc(id).delete();
  }
}
```

### 3. La Interfaz de Usuario (`ui/user_list.dart`)
Aquí el **Agente Director** muestra los resultados del flujo de trabajo.

```dart
import 'package:flutter/material.dart';
import '../agents/data_agent.dart';
import '../models/user_model.dart';

class UserList extends StatelessWidget {
  final DataAgent agent = DataAgent();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("CRUD Ruleta 0310")),
      body: StreamBuilder<List<UserModel>>(
        stream: agent.getUsers(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return CircularProgressIndicator();
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              var user = snapshot.data![index];
              return ListTile(
                title: Text(user.nombre),
                subtitle: Text("${user.numeroTelefono} | ${user.fechaNacimiento}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => agent.deleteUser(user.id!),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showForm(context),
        child: Icon(Icons.add),
      ),
    );
  }

  void _showForm(BuildContext context) {
    // Aquí se implementaría el Modal con los 3 campos solicitados
  }
}
```

---

## Flujo de Trabajo (Workflow)

1.  **Inicio:** El `main.dart` inicializa `Firebase.initializeApp()`.
2.  **Solicitud:** El usuario presiona "Agregar". El **Agente Director** abre el formulario.
3.  **Procesamiento:** Al guardar, los datos pasan al **Agente Archivista**.
4.  **Sincronización:** El Archivista ejecuta el Skill de Firestore. La UI se actualiza automáticamente gracias al `Stream`.

> **Nota de Antigravity:** En una implementación avanzada, usaríamos el `GravityProvider` para inyectar los agentes en el árbol de widgets, permitiendo que cualquier componente acceda a las "Skills" de datos sin pasar referencias manualmente.

<Elicitations message="¿En qué área del proyecto te gustaría profundizar ahora?">
  <Elicitation label="Crear el formulario con validación" query="Muéstrame el código del formulario Dart con validación de campos para Nombre, Fecha y Teléfono." />
  <Elicitation label="Configurar Antigravity avanzado" query="Explica cómo implementar el GravityProvider para gestionar el estado global de los agentes." />
</Elicitations>
