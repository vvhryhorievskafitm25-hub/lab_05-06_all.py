# lab_05-06_all.py
"""
Лабораторна робота №5-6: Веб-додаток на Django (Каталог подій)
Виконано студенткою групи КБ-25: Григор'євська Вікторія
"""

# =====================================================================
# 1. МОДЕЛІ ДАНИХ (models.py)
# =====================================================================
from django.db import models

class Event(models.Model):
    """Модель події для бази даних Django."""
    title = models.CharField(max_length=100, verbose_name="Назва події")
    email = models.EmailField(verbose_name="Контактний Email")
    category = models.CharField(
        max_length=50, 
        choices=[('tech', 'Технології'), ('business', 'Бізнес'), ('art', 'Мистецтво')],
        verbose_name="Категорія"
    )
    description = models.TextField(verbose_name="Опис події")
    created_at = models.DateTimeField(auto_now_add=True, verbose_name="Дата створення")

    def __str__(self):
        return self.title


# =====================================================================
# 2. ФОРМИ ТА СЕРВЕРНА ВАЛІДАЦІЯ (forms.py)
# =====================================================================
from django import forms
from django.core.exceptions import ValidationError

class EventForm(forms.Form):
    """Django-форма з валідацією полів та кастомним методом clean."""
    title = forms.CharField(
        max_length=100, 
        label="Назва події",
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    email = forms.EmailField(
        label="Контактний Email",
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    category = forms.ChoiceField(
        choices=[('tech', 'Технології'), ('business', 'Бізнес'), ('art', 'Мистецтво')],
        label="Категорія",
        widget=forms.Select(attrs={'class': 'form-select'})
    )
    description = forms.CharField(
        widget=forms.Textarea(attrs={'class': 'form-control', 'rows': 3}),
        label="Опис події"
    )

    def clean_title(self):
        """Кастомна валідація: заборона спам-слів у назві події."""
        title = self.cleaned_data.get('title')
        if "спам" in title.lower():
            raise ValidationError("Назва події містить заборонені слова!")
        return title


# =====================================================================
# 3. КОНТРОЛЕРИ / VIEW-ФУНКЦІЇ (views.py)
# =====================================================================
from django.shortcuts import render, redirect
from django.contrib import messages

def event_list_view(request):
    """View для відображення списку подій (Головна сторінка)."""
    # У реальному проекті: events = Event.objects.all()
    events = [
        {"title": "Python Conference 2026", "date": "2026-06-15", "description": "Щорічна конференція розробників.", "capacity": 50},
        {"title": "AI & Cybersecurity Meetup", "date": "2026-06-20", "description": "Обговорення питань безпеки штучного інтелекту.", "capacity": 30}
    ]
    context = {
        "app_name": "каталог подій",
        "total_events": len(events),
        "events": events
    }
    return render(request, 'events/index.html', context)


def add_event_view(request):
    """View для обробки POST/GET запитів форми створення події."""
    if request.method == 'POST':
        form = EventForm(request.POST)
        if form.is_valid():
            # Тут виконується збереження даних у базу (наприклад, Event.objects.create(...))
            messages.success(request, "Подію успішно створено (Django)!")
            return redirect('event_list')
    else:
        form = EventForm()
    
    return render(request, 'events/add_event.html', {'form': form})


# =====================================================================
# 4. ШАБЛОНИ (HTML / DTL) - Структура файлів
# =====================================================================
"""
--- templates/events/base.html ---
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Портал подій{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
        <div class="container">
            <a class="navbar-brand" href="{% url 'event_list' %}">EventPortal</a>
            <div class="navbar-nav">
                <a class="nav-link" href="{% url 'event_list' %}">Головна</a>
                <a class="nav-link" href="{% url 'add_event' %}">Додати подію</a>
            </div>
        </div>
    </nav>

    <div class="container">
        {% if messages %}
            {% for message in messages %}
                <div class="alert alert-success">{{ message }}</div>
            {% endfor %}
        {% endif %}

        {% block content %}{% endblock %}
    </div>
</body>
</html>

--- templates/events/add_event.html ---
{% extends 'events/base.html' %}
{% block title %}Додати подію{% endblock %}
{% block content %}
<h2>Створити нову подію</h2>
<form method="POST" class="mt-3">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit" class="btn btn-primary">Зберегти</button>
</form>
{% endblock %}
"""
