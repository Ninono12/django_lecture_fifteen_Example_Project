# Custom Login and Logout Using Class-Based Views (CBVs)

---

**AuthenticationForm** source code: https://github.com/django/django/blob/c2c7dbb2f88ce8f0ef6d48a61b93866c9926349a/django/contrib/auth/forms.py#L313

## 1. Custom LoginView (inherits from `View`)

```python
# users/views.py
from django.views import View
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login as auth_login
from django.contrib.auth.forms import AuthenticationForm
from django.contrib import messages

class LoginView(View):
    template_name = 'login.html'

    def get(self, request):
        form = AuthenticationForm()
        return render(request, self.template_name, {'form': form})

    def post(self, request):
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            auth_login(request, user)
            messages.success(request, "You are now logged in.")
            return redirect('home')
        messages.error(request, "Invalid username or password.")
        return render(request, self.template_name, {'form': form})
```

---

## 2. Custom LogoutView (inherits from `View`)

```python
# users/views.py
from django.contrib.auth import logout as auth_logout
from django.views import View
from django.shortcuts import redirect
from django.contrib import messages

class LogoutView(View):
    def get(self, request):
        auth_logout(request)
        messages.success(request, "You have been logged out.")
        return redirect('login')
```

---

## 3. URL Patterns

```python
# users/urls.py
from django.urls import path
from .views import LoginView, LogoutView

urlpatterns = [
    path('login/', LoginView.as_view(), name='login'),
    path('logout/', LogoutView.as_view(), name='logout'),
]
```

---

## 4. Login Template (`templates/login.html`)

Use the same template you shared:

```html
<h2>Login</h2>
<form method="POST">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Login</button>
</form>

{% if messages %}
  <ul>
    {% for message in messages %}
      <li>{{ message }}</li>
    {% endfor %}
  </ul>
{% endif %}
```

---

## 5. Protecting Views

Use the `LoginRequiredMixin` in CBVs to protect views (instead of the decorator used in FBVs):

```python
from django.contrib.auth.mixins import LoginRequiredMixin
from django.views.generic import TemplateView

class HomeView(LoginRequiredMixin, TemplateView):
    template_name = 'home.html'
```

Add URL:

```python
# users/urls.py
from .views import HomeView

urlpatterns += [
    path('home/', HomeView.as_view(), name='home'),
]
```

---

## Summary

| Task     | Implementation                                               |
| -------- | ------------------------------------------------------------ |
| Login    | Custom `LoginView` subclassing `View` with manual auth logic |
| Logout   | Custom `LogoutView` subclassing `View` calling `logout()`    |
| Protect  | Use `LoginRequiredMixin` in CBVs                             |
| Template | Use standard Django forms and messages for UX                |

---
