from django.db import models
from django.contrib.auth.models import User

class ServiceRequest(models.Model):
    REQUEST_TYPES = [
    ('install', 'Installation'),
        ('repair', 'Repair'),
        ('maintenance', 'Maintenance'),


    user = models.ForeignKey(User, on_delete=models.CASCADE)
    request_type = models.CharField(max_length=50, choices=REQUEST_TYPES)
    description = models.TextField()
    attachment = models.FileField(upload_to='attachments/', blank=True, null=True)
    status = models.CharField(max_length=20, default='Pending')
    submitted_at = models.DateTimeField(auto_now_add=True)
    resolved_at = models.DateTimeField(blank=True, null=True)

    def __str__(self):
        return f"{self.user.username} - {self.request_type}"
from django import forms
from .models import ServiceRequest

class ServiceRequestForm(forms.ModelForm):
    class Meta:
        model = ServiceRequest
        fields = ['request_type', 'description', 'attachment']
from django.shortcuts import render, redirect
from .models import ServiceRequest
from .forms import ServiceRequestForm

def submit_request(request):
    if request.method == 'POST':
        form = ServiceRequestForm(request.POST, request.FILES)
        if form.is_valid():
            service_request = form.save(commit=False)
            service_request.user = request.user
            service_request.save()
            return redirect('track_requests')
    else:
        form = ServiceRequestForm()
    return render(request, 'submit_request.html', {'form': form})

def track_requests(request):
    requests = ServiceRequest.objects.filter(user=request.user)
    return render(request, 'track_requests.html', {'requests': requests})
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit Request</button>
</form>
<h2>Your Service Requests</h2>
<ul>
    {% for request in requests %}
    <li>
        Type: {{ request.request_type }} | Status: {{ request.status }} | Submitted: {{ request.submitted_at }}
    </li>
    {% endfor %}
</ul>
from django.contrib import admin
from .models import ServiceRequest

admin.site.register(ServiceRequest)
from django.urls import path
from . import views

urlpatterns = [
    path('submit/', views.submit_request, name='submit_request'),
    path('track/', views.track_requests, name='track_requests'),


from django.contrib import admin
from django.urls import path, include

urlpatterns = [
path('admin/', admin.site.urls),
    path('service/', include('customer_service.urls')),


]

