## 🧾 Objective: Register a New User Using a Class-Based View

We’ll use:

* A **CustomUser** model (as you already have from the previous lecture)
* A **ModelForm** for user registration
* A **CreateView**
* A simple **registration template**

---

## 🧱 1. Create a `UserRegistrationForm`

```python
# accounts/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from .models import CustomUser

class UserRegistrationForm(UserCreationForm):
    class Meta:
        model = CustomUser
        fields = ('email', 'full_name', 'password1', 'password2')
```

✅ `UserCreationForm` already includes password validation.

---

## 🧠 2. Create the Registration View (CBV)

```python
# accounts/views.py
from django.urls import reverse_lazy
from django.views.generic.edit import CreateView
from .forms import UserRegistrationForm
from .models import CustomUser

class UserRegisterView(CreateView):
    model = CustomUser
    form_class = UserRegistrationForm
    template_name = 'accounts/register.html'
    success_url = reverse_lazy('login')  # or a custom success page
```

---

## 🌐 3. Add URL Pattern

```python
# accounts/urls.py
from django.urls import path
from .views import UserRegisterView

urlpatterns = [
    path('register/', UserRegisterView.as_view(), name='register'),
]
```

Include it in your main `urls.py` if needed:

```python
# project/urls.py
path('accounts/', include('accounts.urls')),
```

---

## 🖼️ 4. Create the Template: `register.html`

```html
<!-- templates/accounts/register.html -->
<h2>Register</h2>

<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Sign Up</button>
</form>

<p>Already have an account? <a href="{% url 'login' %}">Login here</a>.</p>
```

---

## 🛂 5. Optional: Enable Django’s Built-in Auth URLs

```python
# project/urls.py
from django.contrib.auth import views as auth_views

urlpatterns += [
    path('accounts/login/', auth_views.LoginView.as_view(), name='login'),
    path('accounts/logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

---

## ✅ Test the Flow

* Visit `/accounts/register/`
* Fill the form
* Submit
* User is created and redirected to login

---

## 🔒 Optional Improvements

* Automatically login the user after registration using `form_valid` override
* Add success message via `django.contrib.messages`
* Style the form using Bootstrap

---
