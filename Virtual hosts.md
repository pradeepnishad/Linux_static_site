/var/www/
│
├── mywebsite/
│   └── index.html
│
├── site1/
│   └── index.html
│
└── site2/
    └── index.html

/etc/nginx/
│
├── sites-available/
│   ├── default
│   ├── site1
│   └── site2
│
└── sites-enabled/
    ├── site1 -> ../sites-available/site1
    └── site2 -> ../sites-available/site2


3. Create the bot folder for website

![Nginx installation](images/two-folder.png)


3. Create the first website

![Nginx installation](images/site1.png)


3. Create the second website

![Nginx installation](images/site2.png)


giving permission

![Nginx installation](images/virtualHostPermission.png)


. Configure the first virtual host


![Nginx installation](images/firstVrtualHostConfiguration.png)



. Configure the second virtual host


![Nginx installation](images/secondVrtualHostConfiguration.png)


7. Enable both virtual hosts
Create symbolic links:

![Nginx installation](images/symbolicLinksForBothSites.png)


Disable the default configuration so it doesn't interfere with our test:


![Nginx installation](images/disablingDefault.png)


Realoading and validation

![Nginx installation](images/reloadingAndValidation.png)

curling both sites

![Nginx installation](images/testingBothSItes.png)