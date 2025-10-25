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
Run: `python manage.py runserver`   
Visit: `http://127.0.0.1:8000/api/colleges/`

⚠️  `{"detail": "Authentication credentials were not provided."}`  
means: ➡️ You tried to access a protected API endpoint without including a valid authentication token or login credentials.
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


