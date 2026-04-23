¡Excelente iniciativa! Como desarrollador senior, he diseñado este plan de trabajo estructurado para que tus estudiantes no solo aprendan a programar, sino a entender la arquitectura detrás de una aplicación funcional y moderna utilizando **Flutter** y **Firebase**.

---

## 1. Preparación del Entorno y Estructura
Primero, organizamos el espacio de trabajo en tu sistema de archivos.

* **Ruta:** `xfluttercancino0517/crudfitnnes`
* **Comando de creación:**
    ```bash
    mkdir -p xfluttercancino0517/crudfitnnes
    cd xfluttercancino0517/crudfitnnes
    flutter create .
    ```

---

## 2. Configuración de Firebase (Consola y Librerías)
Para que el proyecto funcione, debemos vincularlo a la consola de Firebase.

### Paso A: Consola Firebase
1.  Ve a [Firebase Console](https://console.firebase.google.com/).
2.  Crea un proyecto llamado `FitnnesCRUD`.
3.  Crea una base de datos **Cloud Firestore** en modo de prueba.
4.  Crea una colección llamada `empleados`.

### Paso B: Integración de Librerías (`pubspec.yaml`)
Para implementar el CRUD, necesitamos las dependencias oficiales. Aquí te muestro cómo incluirlas:

```yaml
dependencies:
  flutter:
    sdk: flutter
  # Librerías esenciales
  firebase_core: ^3.0.0      # Núcleo de Firebase
  cloud_firestore: ^5.0.0    # Base de datos en tiempo real
```

---

## 3. Metodología de Trabajo: "Antigravity" para Estudiantes
Para este proyecto, utilizaremos una simplificación del concepto **Antigravity** (orientado a la agilidad y eliminación de "peso" innecesario en el código), enfocándonos en la **Inyección de Dependencias** y **Separación de Capas**.

### Definición de Agentes y Roles (Flujo de Trabajo)
Para que los estudiantes entiendan el flujo, asignamos "Roles" a los archivos:

| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Data Agent** | `FirestoreService` | Comunicación directa con la base de datos (CRUD). |
| **Logic Agent** | `EmployeeProvider` | Gestión del estado y validación de datos. |
| **UI Agent** | `EmployeeScreen` | Interfaz visual atractiva y entrada de usuario. |

---

## 4. Estructura de Carpetas (Clean Architecture Lite)
```text
lib/
├── config/           # Configuración de colores y temas
├── services/         # Agente de Datos (Firebase)
├── models/           # Definición del objeto Empleado
└── screens/          # Agente de UI (Vistas)
```

---

## 5. Implementación del Código Funcional

### A. El Modelo (Skills: Definición de Datos)
`lib/models/employee_model.dart`
```dart
class Employee {
  String id;
  String nombre;
  int edad;
  String puesto;

  Employee({required this.id, required this.nombre, required this.edad, required this.puesto});

  // Convertir a Map para enviar a Firebase
  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "edad": edad,
    "puesto": puesto,
  };
}
```

### B. El Agente de Datos (CRUD Service)
`lib/services/firestore_service.dart`
```dart
import 'cloud_firestore.dart';

class FirestoreService {
  final CollectionReference employees = FirebaseFirestore.instance.collection('empleados');

  // CREATE
  Future<void> addEmployee(String n, int e, String p) => employees.add({'nombre': n, 'edad': e, 'puesto': p});

  // READ (Stream)
  Stream<QuerySnapshot> getEmployees() => employees.snapshots();

  // UPDATE
  Future<void> updateEmployee(String id, String n, int e, String p) => 
      employees.doc(id).update({'nombre': n, 'edad': e, 'puesto': p});

  // DELETE
  Future<void> deleteEmployee(String id) => employees.doc(id).delete();
}
```

### C. La Interfaz de Usuario (UI Agent con Colores Atractivos)
`lib/screens/home_screen.dart`
Utilizaremos un diseño con tarjetas de colores y botones flotantes.

```dart
import 'package:flutter/material.dart';
import '../services/firestore_service.dart';

class HomeScreen extends StatelessWidget {
  final FirestoreService service = FirestoreService();
  final TextEditingController nameController = TextEditingController();
  final TextEditingController ageController = TextEditingController();
  final TextEditingController jobController = TextEditingController();

  void showForm(BuildContext context, {String? id}) {
    showModalBottomSheet(
      backgroundColor: Colors.grey[100],
      isScrollControlled: true,
      context: context,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 20, right: 20, top: 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nameController, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: ageController, decoration: InputDecoration(labelText: 'Edad'), keyboardType: TextInputType.number),
            TextField(controller: jobController, decoration: InputDecoration(labelText: 'Puesto')),
            SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.deepPurple, foregroundColor: Colors.white),
              onPressed: () {
                if (id == null) {
                  service.addEmployee(nameController.text, int.parse(ageController.text), jobController.text);
                } else {
                  service.updateEmployee(id, nameController.text, int.parse(ageController.text), jobController.text);
                }
                Navigator.pop(context);
              },
              child: Text(id == null ? 'Crear' : 'Actualizar'),
            )
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('FitnnesCRUD - Empleados'), backgroundColor: Colors.deepPurple),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.orangeAccent,
        onPressed: () => showForm(context),
        child: Icon(Icons.add),
      ),
      body: StreamBuilder(
        stream: service.getEmployees(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          return ListView.builder(
            itemCount: snapshot.data!.docs.length,
            itemBuilder: (context, index) {
              var doc = snapshot.data!.docs[index];
              return Card(
                color: Colors.white,
                elevation: 4,
                margin: EdgeInsets.all(10),
                child: ListTile(
                  title: Text(doc['nombre'], style: TextStyle(fontWeight: FontWeight.bold, color: Colors.deepPurple)),
                  subtitle: Text("${doc['puesto']} - ${doc['edad']} años"),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => showForm(context, id: doc.id)),
                      IconButton(icon: Icon(Icons.delete, color: Colors.redAccent), onPressed: () => service.deleteEmployee(doc.id)),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

---

## 6. Flujo de Trabajo Final
1.  **Inicialización:** En el archivo `main.dart`, asegúrate de llamar a `Firebase.initializeApp()`.
2.  **Práctica Guiada:** Los estudiantes deben crear primero el modelo, luego el servicio (Agente de Datos) y finalmente la UI.
3.  **Evaluación:** Verificar que al borrar un empleado en la app, este desaparezca inmediatamente de la consola de Firebase.

Este plan proporciona una estructura profesional, limpia y fácil de seguir para cualquier nivel de aprendizaje en Flutter.
