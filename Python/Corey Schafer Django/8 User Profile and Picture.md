# Python Django Tutorial: Full-Featured Web App Part 8 - User Profile and Picture - YouTube

## Key Insights

- **Introduction**
  - Continue working on the user profile page.
  - Add the ability for users to upload profile pictures.
  - Extend the existing user model to add more fields.
  - Learn how to use Django signals to run specific functions after certain actions.

- **Extending the User Model**
  - Create a `Profile` model to extend the default user model.
  - Add fields for profile pictures and other user information.
  
  ```python
  from django.db import models
  from django.contrib.auth.models import User

  class Profile(models.Model):
      user = models.OneToOneField(User, on_delete=models.CASCADE)
      image = models.ImageField(default='default.jpg', upload_to='profile_pics')
      bio = models.TextField(blank=True)

      def __str__(self):
          return f'{self.user.username} Profile'
  ```

- **Creating a Signal for Profile Creation**
  - Use Django signals to create a profile automatically when a new user is created.
  
  ```python
  from django.db.models.signals import post_save
  from django.contrib.auth.models import User
  from django.dispatch import receiver
  from .models import Profile

  @receiver(post_save, sender=User)
  def create_profile(sender, instance, created, **kwargs):
      if created:
          Profile.objects.create(user=instance)

  @receiver(post_save, sender=User)
  def save_profile(sender, instance, **kwargs):
      instance.profile.save()
  ```

- **Updating Views for Profile Picture Upload**
  - Modify views to handle profile picture uploads.
  
  ```python
  from django.shortcuts import render, redirect
  from django.contrib.auth.decorators import login_required
  from .forms import UserUpdateForm, ProfileUpdateForm

  @login_required
  def profile(request):
      if request.method == 'POST':
          u_form = UserUpdateForm(request.POST, instance=request.user)
          p_form = ProfileUpdateForm(request.POST, request.FILES, instance=request.user.profile)
          if u_form.is_valid() and p_form.is_valid():
              u_form.save()
              p_form.save()
              messages.success(request, f'Your account has been updated!')
              return redirect('profile')
      else:
          u_form = UserUpdateForm(instance=request.user)
          p_form = ProfileUpdateForm(instance=request.user.profile)

      context = {
          'u_form': u_form,
          'p_form': p_form
      }

      return render(request, 'users/profile.html', context)
  ```

- **Creating Forms for User and Profile Update**
  - Create forms to update user and profile information.
  
  ```python
  from django import forms
  from django.contrib.auth.models import User
  from .models import Profile

  class UserUpdateForm(forms.ModelForm):
      email = forms.EmailField()

      class Meta:
          model = User
          fields = ['username', 'email']

  class ProfileUpdateForm(forms.ModelForm):
      class Meta:
          model = Profile
          fields = ['image', 'bio']
  ```

- **Adding Template for Profile Page**
  - Update the template to include profile picture and bio fields.
  
  ```html
  {% extends "blog/base.html" %}
  {% load crispy_forms_tags %}

  {% block content %}
      <div class="content-section">
          <div class="media">
              <img class="rounded-circle account-img" src="{{ user.profile.image.url }}">
              <div class="media-body">
                  <h2 class="account-heading">{{ user.username }}</h2>
                  <p class="text-secondary">{{ user.profile.bio }}</p>
              </div>
          </div>
          <form method="POST" enctype="multipart/form-data">
              {% csrf_token %}
              {{ u_form|crispy }}
              {{ p_form|crispy }}
              <button class="btn btn-outline-info" type="submit">Update</button>
          </form>
      </div>
  {% endblock %}
  ```

- **Conclusion**
  - Users can now upload profile pictures and add bios to their profiles.
  - Extended the user model with a `Profile` model.
  - Implemented Django signals to automate profile creation.
  - Updated views and forms to handle new profile fields.

These steps cover the essential parts of extending the user model, adding profile pictures, and using Django signals effectively. The provided code snippets are essential for setting up and revising the user profile functionality before attending an interview.