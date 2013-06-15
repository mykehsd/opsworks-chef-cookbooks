# AWS OpstWorks PHP set Environment variables. 

# Summary

Currently it is not possible to setup Environment Variables for PHP stacks in 
 Amazon Web Services OpsWorks. This simple Chef recipe to add environment variables 
 to the .htaccess file by using a template.  The .htaccess file is used for the 
 framework [FuelPHP](http://fuelphp.com) but it can be used for other frameworks
 and php projects. Just change the destination. 

#Requirements
* Apache2
* mod_env must be enabled. 

# Version
0.1.0

# MIT LICENSE

Copyright (c) 2013 onema

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Credits
Some parts of this code where taken from the [ace-cookbooks opsworks_app_environment](https://github.com/ace-cookbooks/opsworks_app_environment). Also see [this](https://forums.aws.amazon.com/thread.jspa?threadID=118107).

# Installation
- In your OpsWorks stack settings enable **"Use custom Chef Cookbooks"**
- Select **"Repository type"** = Git (or any repository you choose to use)
- **"Repository URL"** use git@github.com:onema/chef-cookbooks.git or your Fork Url.
- Set the SSH key if you have one, if not this can be left blank as long as the repo is public. 
- Use the following format for the Custom Chef JSON:

```
{ 
    "custom_env": {
        "staging_site": {
            "env_vars" : [ 
                "AWS_ACCESS_KEY_ID qwerYUIOP",
                "AWS_SECRET_KEY 123RTYU890OPakeicj"
            ],
            "database": {
                "dbname": "staging-database-name", 
                "host": "staging-database.abcd1234.us-east-1.rds.amazonaws.com", 
                "user": "my-user-name", 
                "password": "P@s5w0rD",
                "port": "3306"
           },
           "environment": "staging" 
        },
        "production_site": {
            "env_vars" : [ 
                "CACHE_TIME 1234", 
                "SOME_API_KEY nahnah", 
                "ANOTHER_API_KEY hello-monkey!" 
            ],
            "database": {
                "dbname": "production-database-name", 
                "host": "production-database.abcd1234.us-east-1.rds.amazonaws.com", 
                "user": "my-user-name", 
                "password": "P@s5w0rD",
                "port": "3306"
           },
           "environment": "production" 
        }
    }
}
```

The name custom_env is required. The values staging_site and production_site must match your application name.

In the array of ```env_vars``` for each application you can put any number of environment variables. The format is

"KEY value"

in the example above the production site $_SERVER array would look like this:

```php
Array
(
//... 
    [ANOTHER_API_KEY] => hello-monkey!
    [GA_API_CACHE_TIME] => 1234
    [SOME_API_KEY] => nahnah
//... 
)
```

The environment value is not an optional parameter and will be used in the htaccess template to set the correct environment.

**NOTE: THE RECIPE WILL NOT WORK IF YOUR APPLICATION NAME HAS SPACES OR DASHES "-" BETWEEN WORDS. Use underscores "_" to separate words to avoid problems**

- Finally go to **"Layer"** and edit the **"PHP App Server"**
- Under **"Custom Chef recipes"** -> **"Deploy"** add 

``` phpenv::configure ```

- To setup the values for an RDS database (assuming the DB has been added to the correct OpsWorks security group)
use the recipe ```phpenv::rdsconfig```. This recipe will create a db.php in the correct environment and the values 
will be taken from the database values in the custom JSON. 
- To setup the values for a MySQL database created in an OpsWorks layer, you don't need to add the custom database section and 
instead use the recipe ```phpenv::dbconfig``` all the values will be pulled from the deploy[:database] available to the recipe. 

That's it! next time you deploy your application you will have a custom .htaccess file that contains all the environment variables. 

The .htaccess generated by this recipe will look like this:

```
<IfModule mod_rewrite.c>
  RewriteEngine on 
  RewriteCond %{REQUEST_FILENAME} !-f 
  RewriteCond %{REQUEST_FILENAME} !-d 
  RewriteRule ^(.*)$ index.php/$1 [L] 
  
  <IfModule mod_env.c> 
    # Environment Variables for application production_site 
    SetEnv ANOTHER_API_KEY hello-monkey! 
    SetEnv FUEL_ENV production 
    SetEnv GA_API_CACHE_TIME 1234 
    SetEnv SOME_API_KEY nahnah 
  </IfModule> 
</IfModule>
```
