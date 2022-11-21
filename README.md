# reCaptcha
https://www.google.com/recaptcha/admin/
https://pypi.org/project/django-recaptcha/


# login customised form

from django import forms
from django.contrib.auth import (
    authenticate,
    login,
    logout,
    )
from captcha.fields import ReCaptchaField
from captcha.widgets import ReCaptchaV2Checkbox


class UserLoginForm(forms.Form):
    username=forms.CharField()
    password=forms.CharField(widget=forms.PasswordInput)
    captcha = ReCaptchaField(
        public_key='6LdmuiIjAAAAAMTyTJfK1jnstTYjTISZgIy-iAJ9',
        private_key='6LdmuiIjAAAAALmNlSki9aLaJ0To8SXUC_QHiwjR',
    )
    def clean(self,*args, **kwargs):
        username=self.cleaned_data.get('username')
        password=self.cleaned_data.get('password')
        
        if username and password:
            user=authenticate(username=username , password=password)
            if not user:
                raise forms.ValidationError("User not known")
            if not user.check_password(password):
                raise forms.ValidationError("Invalid password")
        return super(UserLoginForm, self).clean(*args,**kwargs)
        
        
#in views.py

def login_view(request):
    next=request.GET.get('next')
    form=UserLoginForm(request.POST or None)
    if form.is_valid():
        username=form.cleaned_data.get('username')
        password=form.cleaned_data.get('password')
        user=authenticate(username=username,password=password)
        login(request, user)
        if next:
            return redirect(next)
        return redirect("/")
    context={
        'form': form   
    }  
    return render(request, "registration/login.html",context)
#in urls.py


from django.urls import path
from . import views

urlpatterns=[
    
    path('login/',views.login_view, name="login"),
    path('logout/',views.logout_view, name="logout")
    
]
            
        
            
