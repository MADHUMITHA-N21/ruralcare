RURAL CARE
# RURAL CARE Project: Backend Implementation using Python, Django, and PostgreSQL

# Install dependencies
# pip install django djangorestframework psycopg2 twilio boto3

# settings.py (Django settings for AWS and PostgreSQL integration)
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'rural_care',
        'USER': 'your_postgres_user',
        'PASSWORD': 'your_postgres_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

# AWS S3 for storage
AWS_ACCESS_KEY_ID = 'your_aws_access_key'
AWS_SECRET_ACCESS_KEY = 'your_aws_secret_key'
AWS_STORAGE_BUCKET_NAME = 'your_bucket_name'
AWS_S3_REGION_NAME = 'your_region'

# Twilio settings
TWILIO_ACCOUNT_SID = 'your_twilio_account_sid'
TWILIO_AUTH_TOKEN = 'your_twilio_auth_token'

# models.py
from django.db import models

class Patient(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
    address = models.TextField()
    phone_number = models.CharField(max_length=15)
    health_condition = models.TextField()
    
    def __str__(self):
        return self.name

class Appointment(models.Model):
    patient = models.ForeignKey(Patient, on_delete=models.CASCADE)
    doctor_name = models.CharField(max_length=100)
    appointment_time = models.DateTimeField()
    
    def __str__(self):
        return f"{self.patient.name} - {self.doctor_name}"

# serializers.py
from rest_framework import serializers
from .models import Patient, Appointment

class PatientSerializer(serializers.ModelSerializer):
    class Meta:
        model = Patient
        fields = '__all__'

class AppointmentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Appointment
        fields = '__all__'

# views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from .models import Patient, Appointment
from .serializers import PatientSerializer, AppointmentSerializer

class PatientView(APIView):
    def get(self, request):
        patients = Patient.objects.all()
        serializer = PatientSerializer(patients, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = PatientSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class AppointmentView(APIView):
    def get(self, request):
        appointments = Appointment.objects.all()
        serializer = AppointmentSerializer(appointments, many=True)
        return Response(serializer.data)

    def post(self, request):
        serializer = AppointmentSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

# urls.py
from django.urls import path
from .views import PatientView, AppointmentView

urlpatterns = [
    path('patients/', PatientView.as_view(), name='patients'),
    path('appointments/', AppointmentView.as_view(), name='appointments'),
]

# Example usage of Twilio for communication (e.g., appointment reminders)
from twilio.rest import Client

def send_appointment_reminder(phone_number, message):
    client = Client(TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN)
    client.messages.create(
        to=phone_number,
        from_="your_twilio_phone_number",
        body=message
    )

# Example: Using AWS S3 to upload patient files
import boto3

def upload_file_to_s3(file_name, bucket, object_name=None):
    s3_client = boto3.client('s3',
                              aws_access_key_id=AWS_ACCESS_KEY_ID,
                              aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
    try:
        s3_client.upload_file(file_name, bucket, object_name or file_name)
    except Exception as e:
        print(f"Error uploading to S3: {e}")

# Optimizing Android apps for offline and low-resource environments
# Implement offline-first design in Android apps by using Room Database for local data storage.
# This ensures the app works without internet and syncs with the backend when connectivity is restored.

# Example: Android Room Database Schema (Java/Kotlin Example)
# @Entity
# data class Patient(
#     @PrimaryKey val id: Int,
#     val name: String,
#     val age: Int,
#     val address: String,
#     val phoneNumber: String,
#     val healthCondition: String
# )

# Example: Data Sync Logic (Pseudocode)
# - Save patient data locally using Room.
# - Use a WorkManager to periodically sync data with the backend when connectivity is available.
#
# val workRequest = OneTimeWorkRequestBuilder<DataSyncWorker>()
#     .setConstraints(Constraints.Builder()
#         .setRequiredNetworkType(NetworkType.CONNECTED)
#         .build())
#     .build()
# WorkManager.getInstance(context).enqueue(workRequest)

# Run Django migrations and start the development server
# python manage.py makemigrations
# python manage.py migrate
# python manage.p

