# 🧠 RESTful API Development using Django REST Framework (DRF)
## 🎯 Intended Learning Outcomes (ILOs):
- Install and configure Django REST Framework (DRF).
- Create serializers for Django models.
- Build CRUD API endpoints using DRF.
- Apply token authentication and permissions to secure endpoints.
### What is a RESTful API?
A **REST (Representational State Transfer)** API provides a structured way for clients and servers to communicate.
**Key Principles:**
- Client–Server: Separation between UI and backend logic
- Stateless: Each request is independent
- Resource-based URLs: `/api/students/`, `/api/colleges/1/`
- Methods: `GET`, `POST`, `PUT/PATCH`, `DELETE`
- Format: Typically JSON
### DRF Setup
1. Install DRF
```
pip install djangorestframework
```
2. Add to Installed Apps (`settings.py)`
```python
INSTALLED_APPS = [
  ...,
  'rest_framework',
  'studentorg',
]
```
3. REST Configuration (`settings.py`)
```python
REST_FRAMEWORK = {
  'DEFAULT_AUTHENTICATION_CLASSES': [
      'rest_framework.authentication.TokenAuthentication',
  ],
  'DEFAULT_PERMISSION_CLASSES': [
      'rest_framework.permissions.IsAuthenticated',
  ],
}
```
### Setup API App
Create an app
```
python manage.py startapp api
```
Add to Installed Apps (`settings.py`)
```python
INSTALLED_APPS = [
    ...,
    'rest_framework',
    'api',
    'studentorg',
]

```
### What are Serializers?
Serializers convert **Django models** to and from **JSON** for API communication.
Create `api/serializers.py`
```python
from rest_framework import serializers
from studentorg.models import College

class CollegeSerializer(serializers.ModelSerializer):
  class Meta:
      model = College
      fields = '__all__'
      
```
In Django REST Framework (DRF), the **serializer** decides what fields are exposed in your API responses. So to restrict (or hide) fields like the **`id`**, you simply modify your **serializer** definition.
Use `fields = ['college_name', 'created_at', 'updated_at']`
## CRUD Endpoints
Using DRF Generic Views in `api/views.py`
```python
from rest_framework import generics
from studentorg.models import College
from .serializers import CollegeSerializer
from rest_framework.permissions import IsAuthenticated, AllowAny

# List all or create new colleges (protected)
class CollegeListCreateAPIView(generics.ListCreateAPIView):
    queryset = College.objects.all()
    serializer_class = CollegeSerializer
    permission_classes = [IsAuthenticated]  # Require login or token

# Retrieve, update, or delete a single college (protected)
class CollegeRetrieveUpdateDestroyAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = College.objects.all()
    serializer_class = CollegeSerializer
    permission_classes = [IsAuthenticated]  # Require login or token
```
URLs in `api/urls.py`:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('colleges/', views.CollegeListCreateAPIView.as_view(), name='college-list-create'),
    path('colleges/<int:pk>/', views.CollegeRetrieveUpdateDestroyAPIView.as_view(), name='college-detail'),
]
```
Add api urls to `projectsite/urls.py`
```python
urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('api.urls')),  # <-- includes all API routes
]
```
## Testing the API
- Run: `python manage.py runserver`.
- Visit: `http://127.0.0.1:8000/api/colleges/`.
![Screen Shot 2025-10-24 at 9.59.30 AM.png](../assets/Screen_Shot_2025-10-24_at_9.59.30_AM_1761271219896_0.png)
⚠️  `{"detail": "Authentication credentials were not provided."}`  means:
➡️ You tried to access a protected API endpoint without including a valid authentication token or login credentials.
## Token Authentication
Step 1: Add Token Authentication
```python
INSTALLED_APPS = [
  ...
  'rest_framework.authtoken',
]
```
Step 2: Migrate
```
python manage.py migrate
```
Step 3: Generate a token manually using the Django shell
```
python manage.py shell
```
Step 4: Then run the following commands:
replace `tester` of the username available in your database
```python
from rest_framework.authtoken.models import Token
from django.contrib.auth.models import User

user = User.objects.get(username='tester')
token, created = Token.objects.get_or_create(user=user)

print(token.key)
```
you'll get your API token, keep it:
```apl
2eed7d016361e0536e61b9fc930398234f76e525
```
to quit Django shell `quit()`
## Test API in Postman
Download and install Postman (desktop app): `https://www.postman.com/`
Create user account.
Run the server every time you test locally.
**Quick Reference**
| Action | Method | URL Example | Needs Body? | Needs Token? |
|---|---|---|---|---|
| **List all** | GET | `http://127.0.0.1:8000/api/colleges/` | ❌ | ✅ |
| **Retrieve one** | GET | `http://127.0.0.1:8000/api/colleges/1/` | ❌ | ✅ |
| **Create** | POST | `http://127.0.0.1:8000/api/colleges/` | ✅ | ✅ |
| **Update** | PUT | `http://127.0.0.1:8000/api/colleges/1/` | ✅ | ✅ |
| **Delete** | DELETE | `http://127.0.0.1:8000/api/colleges/1/` | ❌ | ✅ |   

💡 **Note:**. 
The base URL `http://127.0.0.1:8000/` is used because the API is running **locally on your computer** (the Django development server).
In a **production environment**, replace it with your actual server domain or IP address, such as:
`https://yourdomain.com/api/colleges/`
or `https://psusphere.pythonanywhere.com/api/colleges/`
## Testing List all (GET)
Headers:
- Authorization: Token your_token (e.g. Token 2eed7d016361e0536e61b9fc930398234f76e525)
🎥 [Watch demo video](https://drive.google.com/file/d/115BDbQNtGDvVWafkuDdxPf4N9YP_0Feg/view?usp=drive_link)
## Testing Retrieve one (GET)
Headers:
- Authorization: Token your_token (e.g. Token 2eed7d016361e0536e61b9fc930398234f76e525)
🎥 [Watch demo video](https://drive.google.com/file/d/1YpMHNDW0oY-R1ghH4-WM5wvz5Ksy_Xto/view?usp=drive_link)
## Testing Create (POST)
Headers:
- Content-Type: application/json
- Authorization: Token your_token (e.g. Token 2eed7d016361e0536e61b9fc930398234f76e525)
Body:
Click Body > raw > JSON
```json
{
"college_name": "College of Computing and Information Sciences"
}
```
🎥 [Watch demo video](https://drive.google.com/file/d/1xlT3Jv-ZWa0keqlq2cNEtpnjaqsnFA5h/view?usp=drive_link)
## Testing Update (PUT)
Headers:
- Content-Type: application/json
- Authorization: Token your_token (e.g. Token 2eed7d016361e0536e61b9fc930398234f76e525)
Body:
```json
{
  "college_name": "College of Information and Computing Studies"
}
```
🎥 [Watch demo video](https://drive.google.com/file/d/1rHPVORN5pps4EFzzAJZ4NN44LnVTJW3W/view?usp=drive_link)
💡 **Note:**.
If the url is `http://127.0.0.1:8000/api/colleges/1/` college_name of the record with primary key = 1 will be replaced by "College of Information and Computing Studies"
## Testing Delete (Delete)
Headers:
- Authorization: Token your_token (e.g. Token 2eed7d016361e0536e61b9fc930398234f76e525)
🎥 [Watch demo video](https://drive.google.com/file/d/1oBelY6dWleAWFryBzD4_A9ayMx2JW3O8/view?usp=drive_link)
## Exposing related models through a RESTful API
### 1. Models (Given)
```python
class Program(BaseModel):
    prog_name = models.CharField(max_length=150, verbose_name="Program")
    college = models.ForeignKey(College, on_delete=models.CASCADE)

class Student(BaseModel):
    student_id = models.CharField(max_length=15, verbose_name="Student ID")
    lastname = models.CharField(max_length=25, verbose_name="Last Name")
    firstname = models.CharField(max_length=25, verbose_name="First Name")
    middlename = models.CharField(max_length=25, blank=True, null=True, verbose_name="Middle Name")
    program = models.ForeignKey(Program, on_delete=models.CASCADE)

```
### 2. Serializers
We’ll create:
`ProgramSerializer` (shows program info and its students)
`StudentSerializer` (shows student info with their program)
```python
from studentorg.models import College, Student, Program # <-- update

# Serializer for Student (nested in Program)
class StudentSerializer(serializers.ModelSerializer):
    program_name = serializers.CharField(source='program.prog_name', read_only=True)

    class Meta:
        model = Student
        fields = ['id', 'student_id', 'lastname', 'firstname', 'middlename', 'program', 'program_name']
        read_only_fields = ['program_name']
        
# Serializer for Program with nested students
class ProgramSerializer(serializers.ModelSerializer):
    students = StudentSerializer(many=True, read_only=True, source='student_set')

    class Meta:
        model = Program
        fields = ['id', 'prog_name', 'college', 'students']
        
```
**StudentSerializer** sample output:
```json
[
    {
        "id": 3,
        "student_id": "2025-2-8096",
        "lastname": "Johnson",
        "firstname": "Garrett",
        "middlename": "Schmidt",
        "program": 6,
        "program_name": "Bachelor of Science in Accountancy"
    },
    {
        "id": 6,
        "student_id": "2024-5-416",
        "lastname": "Solis",
        "firstname": "Christopher",
        "middlename": "White",
        "program": 4,
        "program_name": "Bachelor of Science in Civil Engineering"
    },
]
```
**ProgramSerializer** sample output:
```json
[
    {
        "id": 3,
        "prog_name": "Bachelor of Science in Psychology",
        "college": 2,
        "students": [
            {
                "id": 9,
                "student_id": "2020-1-7652",
                "lastname": "Chen",
                "firstname": "Jason",
                "middlename": "Montgomery",
                "program": 3,
                "program_name": "Bachelor of Science in Psychology"
            },
            {
                "id": 11,
                "student_id": "2023-7-2537",
                "lastname": "Holloway",
                "firstname": "Tracey",
                "middlename": "Swanson",
                "program": 3,
                "program_name": "Bachelor of Science in Psychology"
            },
        ]
	}
]
```
### 3. Views
Add this in `api/views.py`
```python
from studentorg.models import College, Program, Student # <-- update
from .serializers import CollegeSerializer, ProgramSerializer, StudentSerializer # <-- update



# List all programs or create one (protected)
class ProgramListCreateAPIView(generics.ListCreateAPIView):
    queryset = Program.objects.all()
    serializer_class = ProgramSerializer
    permission_classes = [IsAuthenticated]  # Require token/login

# Retrieve, update, delete a program (protected)
class ProgramRetrieveUpdateDestroyAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Program.objects.all()
    serializer_class = ProgramSerializer
    permission_classes = [IsAuthenticated]

# List all students or create one (protected)
class StudentListCreateAPIView(generics.ListCreateAPIView):
    queryset = Student.objects.all()
    serializer_class = StudentSerializer
    permission_classes = [IsAuthenticated]

# Retrieve, update, delete a student (protected)
class StudentRetrieveUpdateDestroyAPIView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Student.objects.all()
    serializer_class = StudentSerializer
    permission_classes = [IsAuthenticated]
```
### 4. URLs
Add this in `api/urls.py`
```python
# Program endpoints
path('programs/', views.ProgramListCreateAPIView.as_view(), name='program-list-create'),
path('programs/<int:pk>/', views.ProgramRetrieveUpdateDestroyAPIView.as_view(), name='program-detail'),

# Student endpoints
path('students/', views.StudentListCreateAPIView.as_view(), name='student-list-create'),
path('students/<int:pk>/', views.StudentRetrieveUpdateDestroyAPIView.as_view(), name='student-detail'),
```
### 📘  **Program Endpoints**
| **Method** | **Endpoint** | **Description** |
|---|---|---|
| **GET** | `http://127.0.0.1:8000/api/programs/` | Get a list of all programs |
| **POST** | `http://127.0.0.1:8000/api/programs/` | Create a new program *(requires `college` ID and authentication)* |
| **GET** | `http://127.0.0.1:8000/api/programs/<id>/` | Get details of a specific program and its related college |
| **PUT** | `http://127.0.0.1:8000/api/programs/<id>/` | Update an existing program |
| **DELETE** | `http://127.0.0.1:8000/api/programs/<id>/` | Delete a program |   

### 🎓  **Student Endpoints**
| **Method** | **Endpoint** | **Description** |
|---|---|---|
| **GET** | `http://127.0.0.1:8000/api/students/` | Get a list of all students and their associated program |
| **POST** | `http://127.0.0.1:8000/api/students/` | Create a new student *(requires `program` ID and authentication)* |
| **GET** | `http://127.0.0.1:8000/api/students/<id>/` | Get details of a specific student (includes program + college info) |
| **PUT** | `http://127.0.0.1:8000/api/students/<id>/` | Update a student’s record |
| **DELETE** | `http://127.0.0.1:8000/api/students/<id>/` | Delete a student |   

## Best Practice in Coding and Designing APIs
**Design and Structure**
1. Follow RESTful conventions
Use **nouns** for resources, not verbs.
- ✅ `/users` → get users
- ❌ `/getUsers`  
Use **plural** resource names: `/students`, `/books`, `/colleges`.
2. Version your API
Add versioning in the URL (or headers) to avoid breaking changes later.
```apl
/api/v1/students

```
**Data and Responses**
1. Use consistent response formats (always return structured JSON)
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "name": "Marcus Aurelius"
  }
}
```
2. Handle errors gracefully
```json
{
  "status": "error",
  "message": "College not found"
}

```
Use proper HTTP status codes:
- 200: OK
- 201: Created
- 400: Bad Request
- 404: Not Found
- 500: Internal Server Error
**Security**
1. Use authentication and authorization
Implement JWT (JSON Web Tokens) or OAuth2 for secure access
Protect endpoints that modify or delete data
2. Validate input data
Prevent SQL injection and data corruption
Use serializers or validation libraries (e.g. Django REST Framework's Serializer)
**Performance**
1. Pagination for large data; instead of returning all records
```apl
GET /students?page=1&limit=20

```
Optimize database queries  

**Maintainability**
1. Use proper naming conventions
2. Document your API (e.g. Postman collections)
3. Use environment variables (store secrets and config in .env file)
