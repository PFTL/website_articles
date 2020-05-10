# Using Cookiecutter to Generate Project Templates

If there is a tool I regret not discovering earlier, that should be Cookiecutter. It is a Python program that allows you to generate projects based on templates. For example, imagine every day you enter to the lab, you create a new folder with the date. Then you add an empty file to keep track of what you do, you also add a folder for data and a folder for scripts to analyse the data. All this manual work is error prone, boring and can lead to finding shortcuts that leave you with a broken structure. 

Cookiecutter allows you to streamline a lot of steps. It allows you to develop complex templates and distribute them over github, bitbucket, or locally. And then the command line tool allows you to start new projects from those templates. If you are working in a team, nothing better that everyone starting off from the same structure, and grow from there. Cookiecutter is so flexible that it can be used to streamline the creation of a new program, a paper, or daily administration tasks. In this article we are going to show a general overview of what can be achieved. 

## Installing
Cookiecutter is one of those tools that is worth installing in the base environment of the computer. This will enable us to create new projects independently of the virtual environment in which we would like to work. 

```bash
pip install cookiecutter 
```

And we are done. 

## Getting Started
To get started, we can use a template which is available for the creation of new Python packages, and which is maintained by one of the core developers of Cookiecutter itself:

```bash
cookiecutter https://github.com/audreyr/cookiecutter-pypackage.git
```

When you run the command above, you will be asked several questions regarding what you are creating. It doesn't really matter what you answer, but you can see that once it is done, you will have a folder with the name you provided, and with plenty of files inside. These files have the information you supplied during the creation, such as your name, the name of the project, the license you chose, etc. 

Cookiecutter is based on a templating engine called Jinja2, which is also common for web applications. When we tell cookiecutter to use a Github repository, it will use it as a template, therefore we can go directly to the repository to see how things work. If we head to the [Cookiecutter PyPackage repository](https://github.com/audreyr/cookiecutter-pypackage), we will find the same files that we have on our computer, but some are strage. For example, there is a folder called ``{{cookiecutter.project_slug}}``. If you are familiar with Jinja, you will realize this is the way to use it. If you are not familiar with Jinja, it can be puzzling at the beginning. 

The ``project_slug`` was one of the questions we were asked when creating the project. If we explore the folder, we will see that we actually got a folder with that name, without a sign of Cookiecutter anywhere. But that is not all. If we explore the [Readme file](https://github.com/audreyr/cookiecutter-pypackage/blob/master/%7B%7Bcookiecutter.project_slug%7D%7D/README.rst), we can see that there are plenty of template tags being used. We can see that the ``project_name`` is there, and also that there is some control flow with ``if`` statements. If you compare how the base template looks like and what you got on your computer, you will be able to understand what is going on. 

But don't despair, because we will see step by step how the templates work. If you want to see what else can Cookiecutter do, you can quickly head to [Github topics](https://github.com/topics/cookiecutter) and check what templates are available. You will see that there are even templates to get started with programming projects in other languages besides Python. Templates for Django websites, for Data Science projects, and more. However, templates can be very useful even for oneself, or for small groups of people. 

## Getting Started with Templates
We are going to develop a template to be able to start acquiring data in the lab, keeping the data separated from the results, and adding few files to keep track of the progress done. Let's start by creating a new folder, called **cookiecutter-lab**