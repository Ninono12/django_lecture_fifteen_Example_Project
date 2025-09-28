## ✅ Why Override the User Model?

* Use **email** as the login identifier (instead of `username`)
* Add custom fields (`phone_number`, `full_name`, etc.)
* Gain full control over user behavior and authentication

---

## 🔧 Steps to Override the User Model in Django

---

### 🧱 1. Create a Custom User Model

#### ✅ `accounts/models.py`

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.db import models

class CustomUserManager(BaseUserManager):
    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError("Email must be provided")
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        
        if not extra_fields.get('is_staff'):
            raise ValueError('Superuser must have is_staff=True.')
        if not extra_fields.get('is_superuser'):
            raise ValueError('Superuser must have is_superuser=True.')
        
        return self.create_user(email, password, **extra_fields)

class CustomUser(AbstractBaseUser, PermissionsMixin):
    email = models.EmailField(unique=True)
    full_name = models.CharField(max_length=100)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['full_name']

    def __str__(self):
        return self.email
```

---

### ⚙️ 2. Register the Custom User in Settings

#### ✅ `settings.py`

```python
AUTH_USER_MODEL = 'accounts.CustomUser'
```

---

### 📁 3. Create and Apply Migrations

```bash
python manage.py makemigrations accounts
python manage.py migrate
```

⚠️ Important: Do this before the first migration of `auth` or `admin`!

---

### 🛠️ 4. Register in Admin (optional)

#### ✅ `accounts/admin.py`

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import CustomUser

class CustomUserAdmin(UserAdmin):
    model = CustomUser
    list_display = ('email', 'full_name', 'is_staff', 'is_active')
    list_filter = ('is_staff', 'is_active')
    search_fields = ('email',)
    ordering = ('email',)
    fieldsets = (
        (None, {'fields': ('email', 'full_name', 'password')}),
        ('Permissions', {'fields': ('is_staff', 'is_active', 'is_superuser', 'groups', 'user_permissions')}),
    )
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'full_name', 'password1', 'password2', 'is_staff', 'is_active')}
        ),
    )

admin.site.register(CustomUser, CustomUserAdmin)
```

---

### 🧪 5. Create Superuser with Email

```bash
python manage.py createsuperuser
# You will be asked for email and full_name instead of username
```

---

## 🧠 Summary

| What You Did             | Purpose                                 |
| ------------------------ | --------------------------------------- |
| Created `CustomUser`     | To use email instead of username        |
| Made `CustomUserManager` | To control user creation logic          |
| Set `AUTH_USER_MODEL`    | To tell Django to use your custom model |

---
