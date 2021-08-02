# Introduction 
This project is to demonstrate how to containerize Django App with a separate Database (PostgreSQL) and deploy it to Azure.


# Getting Started
You can follow the instructions here: https://code.visualstudio.com/docs/python/tutorial-django.
Goal for this tutorial is provide a step by step from setting up your environment with Python to integrate CI/CD pipeline for future deployments.


# Requirements:

<h3>1.	Install Python and create a virtual environment </h3><br>
    a. Install Python <br>
    b. Run command -  pip install virtualenv <br>
    c. Run command - python -m venv <name of your virtual env> <br>
    d. Select your interpreter from VS Code <br>
    e. Inside your virtual environment run command - python -m pip install django <br>
    f. Run command - django-admin startproject <name of project> <br><p>
    

<h3>2.	Create App </h3><br>
    a. Inside your project folder Run command - python manage.py startapp <name of the app> <br>
    b. Open settings.py and look for the INSTALLED_APPS section and add the name of your app at the end <br>
    c. Run command - python manage.py migrate <br>
    d. Run command - python manage.py runserver <br>
    e. Validate app on your browser in http://127.0.0.1:8000 <br>
    d. Modify <app>/models.py to match the following code, which creates the database models for the blog and save <br>

        from django.db import models
        from django.contrib.auth.models import User


        STATUS = (
            (0,"Draft"),
            (1,"Publish")
        )

        class Post(models.Model):
            title = models.CharField(max_length=200, unique=True)
            slug = models.SlugField(max_length=200, unique=True)
            author = models.ForeignKey(User, on_delete= models.CASCADE,related_name='blog_posts')
            updated_on = models.DateTimeField(auto_now= True)
            content = models.TextField()
            created_on = models.DateTimeField(auto_now_add=True)
            status = models.IntegerField(choices=STATUS, default=0)

            class Meta:
                ordering = ['-created_on']

            def __str__(self):
                return self.title


<br>e. Run command - python manage.py makemigrations <br>
    f. run command - python manage.py migrate <br> <p>


<h3>3. Create Admin Site </h3> <br>
    a. Run command - python manage.py createsuperuser and you will be prompted for username, email password. You can always change them later. <br>
    b. Run command - python manage.py runserver <br>
    c. Validate authentication prompt by going to http://127.0.0.1:8000/admin <br><p>

<h3>4. Adding Models to the Administration Site </h3><br>
    a. Open your <app>/admin.py and register the Post model there as follows and save it:

        from django.contrib import admin
        from .models import Post 

        admin.site.register(Post)


<p> b. Refresh page and you should see the post Model <br>
    c. Create your first post to validate <br>
    d. Open admin.py file and replace it with the following code to make your post more efficient <br>

        from django.contrib import admin
        from .models import Post

        class PostAdmin(admin.ModelAdmin):
            list_display = ('title', 'slug', 'status','created_on')
            list_filter = ("status",)
            search_fields = ['title', 'content']
            prepopulated_fields = {'slug': ('title',)}
        
        admin.site.register(Post, PostAdmin)

    
<p> e. Refresh your page and you should see more information about your posts <br>


<h3>5. Build your views </h3> <br>
    a. Open the <app>/views.py file and start coding: <br>

        from django.views import generic
        from .models import Post

        class PostList(generic.ListView):
            queryset = Post.objects.filter(status=1).order_by('-created_on')
            template_name = 'index.html'

        class PostDetail(generic.DetailView):
            model = Post
            template_name = 'post_detail.html'

<p>
<h3>6. Adding URL patterns for Views </h3> <br>
    a. Create urls.py file inside your app folder and add the following code: <br>

        from . import views
        from django.urls import path

        urlpatterns = [
            path('', views.PostList.as_view(), name='home'),
            path('<slug:slug>/', views.PostDetail.as_view(), name='post_detail'),
        ]

<p>
    b. Open yourproject/urls.py and add the following code in the URL patterns: <br>

        from django.contrib import admin
        from django.urls import path, include

        urlpatterns = [
            path('admin/', admin.site.urls),
            path('', include('blogapp.urls')),
        ]

<p>
    c. Open the project's settings.py file and just below BASE_DIR add the route to the template directory as follows: <br>

        TEMPLATES_DIRS = os.path.join(BASE_DIR,'templates')

        (Note: you will need to import os.path in the settings.py)

<p>
    d. In setings.py scroll to the TEMPLATES, and add TEMPLATE_DIRS in the DIRS

        TEMPLATES = [
            {
                'BACKEND': 'django.template.backends.django.DjangoTemplates',
                'DIRS': [TEMPLATES_DIRS],
                'APP_DIRS': True,
                'OPTIONS': {
                    'context_processors': [
                        'django.template.context_processors.debug',
                        'django.template.context_processors.request',
                        'django.contrib.auth.context_processors.auth',
                        'django.contrib.messages.context_processors.messages',
                    ],
                },
            },
        ]

<p>
    e. Create a folder called templates inside the app folder, and create base.html, index.html, post_detail.html, and sidebar.html <br>
        (look for sample code inside blogapp/templates)<br>
<p>
<h3>7. Containerize your app </h3><br>
    a. Create container registry in Azure portal <br>
    b. Install Docker extension from VS Code and Azure tools <br>
    c. Open command palette and search for Docker:Add Docker Files to Workspace <br>
    d. Specify Python <br>
    e. Select the port 8000 <br>
    f. Move your Dockerfiles to inside your project folder (Dockerfile, .dockerignore, docker-compose.debug.yml, docker-compose.yml) <br>
    g. In your Django Project's settings.py, add the root URL to which you intent to deploy the app to the ALLOWED_HOSTS list:<br>

        ALLOWED_HOSTS = [
            "localhost",
            "127.0.0.1"
          
        ]

<p>
    h. Edit Dockerfile with following code:<br>

    FROM python:3.6-slim

    ENV PYTHONDONTWRITEBYTECODE 1
    ENV PYTHONUNBUFFERED 1
    RUN mkdir /blogapp
    WORKDIR /blogapp
    RUN pip install --upgrade pip
    COPY requirements.txt /blogapp/

    RUN pip install -r requirements.txt
    COPY . /blogapp/

    EXPOSE 8000

    CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

<p>
<h3>4. Build and Test your Image</h3> <br>
    a. From your project folder, run command - docker build -t giveanametoyourtag ex: django-app <br>
    b. Run command - docker run -p 8000:8000 django-app <br>
    c. Validate your app is running on localhost <br>

<p>
<h3>5. Login to Container registry </h3> <br>
    a. Run az acr login -n youracrname <br>
    b. Adjust the docker compose file in the image field to the following: <br>

        services:
        django:
            image: <acrname>.azurecr.io/django-app:latest
            build:
            context: .
            dockerfile: ./Dockerfile
            ports:
            - 8000:8000
<p>
    c. Run command docker-compose up and validate is still visible locally. Make sure to run this from the same folder where the Dockerfile is. <br>
    d. Push the image to the registry by right click on the docker image and push or run command - docker push yourcontainerregistryloginserver/django-app:latest <br>
    e. Validate repository image is visible on Azure. <br>

<p>

<h3> 6. Deploy image using CI/CD Pipeline with Github Actions (Preferred Method) </h3><br>
        a. Create a directory in the top folder or your repo called ARMTemplates <br>
        b. Create file inside ARMTemplates called webApp.json you can use templates in https://github.com/oortizmcp/django/tree/main/ARMTemplates as an examples. <br>
        c. In the .gitignore file, add the .vscode and the *venv/ to avoid pushing these files into the repo. <br>
        d. Commit your code to your repo in Github. <br>
            1) git status <br>
            2) git add . <br>
            3) git commit -m 'your message in the commit' <br>
            4) git push <br>
        e. Go to Settings Tab- Secrets and add the following repository secrets:<br>
            1) AZURE_SP - in azure portal or in commandline, run az ad sp create-for-rbac --name "yourspnnamehere" --role contributor --scopes /subscriptions/yoursubid --sdk-auth and copy the json output into the value for this secret <br>
            2) REGISTRY_PASSWORD - go to Access Keys in your AZ Container registry and enable the Admin User, copy the password and paste it in the value for the registry passsword <br>
            3) REGISTRY_URL - copy the registry Login Server value <br>
            4) REGISTRY_USERNAME - copy the Username value <br>
        f. Go to Actions tab and click on new Workflow- Set up a workflow yourself <br>
        g. Go to https://github.com/oortizmcp/django/blob/main/.github/workflows/main.yml click in the Raw button and paste it in your new workflow <br>
        h. Edit the env section and adjust it as necessary according to your needs. <br>
            (note: The ${{ secrets.REGISTRY_URL}} is making reference to the secrets you created in step 6e) <br>
        i. in Section Build and push the image tagged with the git commit hash, make sure to change to directory where your project is example: cd yourdjangoprjtfolder <br>
        j. Save it and see how the pipeline runs. <br>
<p>

    *Note: Webapp might not reflect the changes immediately after your pipeline runs you might need to restart the webapp from the portal. Also, remember to add the Azure website into the ALLOWED_HOSTS section in the settings.py file.

<h4> Make a change on color or text on base.html, push changes to repo and validate changes. (Note: Webapp will refresh when pipeline finish running and it may take a few minutes to reflect changes.) </h4>



 <h3>*6. Deploy image into Azure WebApp using CI/CD pipeline (Azure Devops)</h3> <br>
    a. Go to Azure Portal and create a webApp with the following Specs:<br>
    <ul>Project Details
        <li>web app name (note: has to be unique) </li>
        <li>Publish: Docker Container</li>
        <li>Operating System: Linux </li>
    </ul>
    <ul>Docker Details:
        <li>Options: Single Container </li>
        <li>Image Source: Azure Container Registry </li>
        <li> Select Registry, Image and Tag you created earlier</li>
    </ul>
    
<p>
    b. Go to Azure Devops project repo. <br>
    c. Select Setup a Build or Create a new Pipeline from the Pipeline blade. <br>
    d. In the Configure your pipeline section, select Docker (build and push an image to Azure Container Registry). <br>
    e. Select your Subscription. <br>
    f. Select your Container registry. <br>
    g. Review your code and change tag in the variable section from '$(Build.BuildId) to 'latest'. <br>

* Note: This will always look at the latest, preferred method to deploy is github actions. Please refer to documentation for best practices here: https://docs.microsoft.com/en-us/azure/app-service/deploy-best-practices#continuously-deploy-containers 

<h4> Make a change on color or text on base.html, push changes to repo and validate changes. (Note: Webapp will refresh when pipeline finish running and it may take a few minutes to reflect changes.) </h4>