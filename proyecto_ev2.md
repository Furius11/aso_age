# Proyecto de la 2ª Evaluación: Planificación de un Dominio para un Centro Educativo

## Descripción del Proyecto
El proyecto consiste en planificar y diseñar un dominio para un centro educativo que imparte ciclos formativos de la familia de Informática y Comunicaciones (ASIR, SMR, DAM, DAW). El objetivo es establecer una estructura de dominio que cumpla con los requisitos proporcionados, justificando las decisiones tomadas en cada paso.

---

## 1. Estructura de Unidades Organizativas (OU) y Grupos

### 1.1. Estructura de Unidades Organizativas (OU)
La estructura de unidades organizativas se diseña en función de los ciclos formativos y los cursos, con el fin de facilitar la administración de usuarios, grupos y políticas. La estructura propuesta es la siguiente:

- **Dominio Principal**: `centroeducativo.local`
  - **OU Alumnos**
    - **OU ASIR**
      - **OU ASIR1** (Primer curso)
      - **OU ASIR2** (Segundo curso)
    - **OU SMR**
      - **OU SMR1** (Primer curso)
      - **OU SMR2** (Segundo curso)
    - **OU DAM**
      - **OU DAM1** (Primer curso)
      - **OU DAM2** (Segundo curso)
    - **OU DAW**
      - **OU DAW1** (Primer curso)
      - **OU DAW2** (Segundo curso)
  - **OU Profesores**
  - **OU Aulas**
    - **OU Aula1**
    - **OU Aula2**
    - ...
    - **OU AulaN** (Dependiendo del número de aulas disponibles)

**Justificación**:
- Separar a los alumnos por ciclos y cursos facilita la aplicación de cualquier cosa específica para cada grupo.
- Los profesores se agrupan en una OU independiente, ya que pueden impartir clases en cualquier ciclo y curso.
- Las aulas se organizan en una OU para gestionar los equipos y permisos de acceso.

### 1.2. Estructura de Grupos
Se crearán grupos de seguridad para gestionar los permisos y políticas de acceso. La estructura de grupos será la siguiente:

- **Grupos de Alumnos**:
  - `ASIR1_Alumnos`, `ASIR2_Alumnos`
  - `SMR1_Alumnos`, `SMR2_Alumnos`
  - `DAM1_Alumnos`, `DAM2_Alumnos`
  - `DAW1_Alumnos`, `DAW2_Alumnos`

- **Grupos de Profesores**:
  - `Profesores`

- **Grupos de Aulas**:
  - `Aula1_Equipos`, `Aula2_Equipos`, ..., `AulaN_Equipos`

**Justificación**:
- Los grupos de alumnos permiten aplicar políticas específicas por curso y ciclo.
- El grupo de profesores facilita la gestión de permisos y excepciones.
- Los grupos de aulas permiten controlar el acceso a los equipos y recursos compartidos.

---

## 2. Automatización del Alta de Alumnos

### 2.1. Uso del Comando `Import-Csv`
Para automatizar el alta de alumnos, se utilizará el comando `Import-Csv` de PowerShell. Este comando permite leer un archivo CSV con los datos de los alumnos y crear cuentas de usuario en el dominio.

**Script de ejemplo**:
```powershell
# Importar el archivo CSV
$alumnos = Import-Csv -Path "C:\ruta\alumnos.csv"

# Recorrer cada fila del CSV
foreach ($alumno in $alumnos) {
    # Crear el nombre de usuario 
    $username = "$($alumno.Nombre).$($alumno.Apellido)"
    
    # Crear la cuenta de usuario en el dominio
    New-ADUser `
        -Name "$($alumno.Nombre) $($alumno.Apellido)" `
        -GivenName $alumno.Nombre `
        -Surname $alumno.Apellido `
        -SamAccountName $username `
        -UserPrincipalName "$username@centroeducativo.local" `
        -Path "OU=$($alumno.Curso),OU=$($alumno.Ciclo),OU=Alumnos,DC=centroeducativo,DC=local" `
        -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) `
        -Enabled $true
}
```
## Justificación

El uso de PowerShell permite automatizar el proceso de creación de usuarios, reduciendo errores y ahorrando tiempo.

El archivo CSV debe contener columnas como `Nombre`, `Apellido`, `Ciclo` y `Curso` para que el script funcione correctamente.

---

## 2.2. Automatización de la Creación de Carpetas Personales
Cada alumno tendrá una carpeta personal en el servidor. Para automatizar este proceso, se puede extender el script anterior para crear las carpetas y asignar permisos.

**Extensión del script**:
```powershell
foreach ($alumno in $alumnos) {
    # Crear la carpeta personal
    $carpetaPersonal = "C:\CarpetasPersonales\$username"
    New-Item -Path $carpetaPersonal -ItemType Directory

    # Asignar permisos a la carpeta
    $acl = Get-Acl $carpetaPersonal
    $permisos = "$username", "FullControl", "Allow"
    $regla = New-Object System.Security.AccessControl.FileSystemAccessRule($permisos)
    $acl.SetAccessRule($regla)
    Set-Acl -Path $carpetaPersonal -AclObject $acl
}
```

## Justificación

- La creación automática de carpetas personales asegura que cada alumno tenga su espacio de almacenamiento.
- Los permisos se asignan para que solo el alumno y los administradores tengan acceso a la carpeta.

---

## 3. Estructura de Carpetas Compartidas

### 3.1. Carpetas Compartidas por Grupo
Cada grupo de alumnos tendrá una carpeta compartida. Estas carpetas se crearán en el servidor y se asignarán permisos a los grupos correspondientes.

**Estructura de carpetas**:
- `C:\CarpetasCompartidas\ASIR1`
- `C:\CarpetasCompartidas\ASIR2`
- `C:\CarpetasCompartidas\SMR1`
- `C:\CarpetasCompartidas\SMR2`
- `C:\CarpetasCompartidas\DAM1`
- `C:\CarpetasCompartidas\DAM2`
- `C:\CarpetasCompartidas\DAW1`
- `C:\CarpetasCompartidas\DAW2`

**Permisos**:
- Los alumnos de cada grupo tendrán permisos de lectura y escritura en su carpeta compartida.
- Los profesores tendrán permisos de control total en todas las carpetas compartidas.

**Justificación**:
- Las carpetas compartidas fomentan la colaboración entre los alumnos.
- Los permisos se asignan en función de los grupos para garantizar el acceso adecuado.

---

## 4. Políticas del Dominio

### 4.1. Directivas de Configuración de Seguridad
Se establecerán políticas de seguridad para proteger el dominio y los recursos. Algunas de las políticas clave son:

- **Restricción del uso de CMD**:
  - Los alumnos no podrán ejecutar el símbolo del sistema (`cmd.exe`).
  - Excepción: Los profesores tendrán acceso completo al CMD.

- **Política de contraseñas**:
  - Longitud mínima: 8 caracteres.
  - Complejidad habilitada (mayúsculas, minúsculas, números y caracteres especiales).
  - Caducidad de contraseñas: 90 días.

- **Bloqueo de cuentas**:
  - Número máximo de intentos fallidos: 3.
  - Tiempo de bloqueo: 30 minutos.

- **Auditoría de eventos**:
  - Habilitar auditoría de inicio de sesión y acceso a recursos.

**Justificación**:
- Estas políticas maximizan la seguridad del dominio y protegen los recursos del centro educativo.
- Las excepciones para los profesores garantizan que puedan realizar tareas administrativas sin restricciones.
