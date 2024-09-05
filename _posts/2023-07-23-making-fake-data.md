---
layout: single
title:  "Generating fake user profiles using the python Faker class"
date:   2023-07-23 18:45:00 +1000
tags: python Faker 
seo_tags: python Faker fake-data
seo_title: "Generating fake user profiles using the python Faker class"
seo_description: "Using the python Faker package that generates fake data is useful for testing purposes when you need to use dummy data."
excerpt: "This post describes how to use the python Faker that generates fake data and is useful for testing purposes when you need to use dummy data."
---


![Building blocks image]({{ site.url }}{{ site.baseurl }}/assets/images/fake.jpg)
{: .full}


## Introduction

In many cases, when developing applications or testing functionalities, it is necessary to have a large amount of realistic-looking user data. Manually creating this data can be time-consuming and inefficient. Luckily, Python provides a powerful package called [Faker](https://faker.readthedocs.io/en/master/) that allows us to generate fake user data quickly and easily.

Bear in mind that the data are generated will not be perfect [distributions, values, etc] unless we tune the standard output code. In this case I will work through how to generate content and reference that to make new fields. For example, genrate user names and then use that to generate email addresses.

### What is the Faker Package?

Faker is a Python package for generating fake data, with a large number of providers for generating different types of data, such as people, credit cards, dates/times, cars, phone numbers, etc. Many of the included and community providers are even localized for different regions [where bank accounts, phone numbers, etc are different].

### Installation

Installing Faker is no different than installing any other Python package using pip. You can use either of the following commands to install Faker.

```shell
pip install faker
python -m pip install faker
```

### Basic Usage

Once installed, you can start using the Faker package in your Python code. Here's an example of how to generate a fake name. First of all, initialise the Faker Class and then seed the faker. We will also set a seed so if you use the same seed you will get the same output. This is useful for testing.

```python
from faker import Faker

Faker.seed(0)

fake = Faker()
name = fake.name()

print(name)
```

This will output a randomly generated full name like "Dickie Small" or "Amanda Hugginkiss".

### Generating a more complex user profile

Let's say we want to generate multiple user profiles with various attributes such as name, address, email address, and phone number. We can accomplish this by utilizing the different providers offered by the Faker package. We also want to localise these to our region. In this case, I will use the `en-AU` locale.

I also want to include a field to denote if a person identifies as a First Nations Australian. This is not part of the `Faker` class so we will do this manually using the `random` module. We also want to attribute the weights of the population to the data. In this case, we will use the 2016 census data from the Australian Bureau of Statistics.

Here is a function to create First Nations status using the ABS population distribution.

``` python 
def create_atsi():
    atsi = ['Aboriginal but Not Torres Strait Islander Origin',
            'Torres Strait Islander but not Aboriginal Origin',
            'Both Aboriginal and Torres Strait Islander Origin',
            'Neither Aboriginal nor Torres Strait Islander Origin',
            'Not stated or Inadequately described'
            ]
    return random.choices(atsi, weights=[0.027, 0.008, 0.007, 0.85, 0.108])[0]

```



Now let's use this to create a function that allows us to generate fake Australian user profiles. In this example we create a function and pass in a sex parameter. This will allow us to specify male or female profiles. We will use these with the fake list of domains to use for the email addresses and usernames.

```python

```python
def create_user_australian(sex):
    fake = Faker('en_AU')

    if sex == 'M':
        first_name = fake.first_name_male()
        last_name = fake.last_name_male()
    elif sex == 'F':
        first_name = fake.first_name_female()
        last_name = fake.last_name_female()

    date_of_birth = fake.date_of_birth(minimum_age=24, maximum_age=72).strftime('%Y-%m-%d'),
    email = first_name.lower() + "." + last_name.lower() + "@" + random.choice(domains)
    atsi_status = create_atsi()
    sex = sex
    password = fake.password(length=8)

    # Generating username by concatenating first name and last name
    username = f"{last_name.lower()}{first_name[:2].lower()}"

    user_data = {
        'first_name': first_name,
        'last_name': last_name,
        'date_of_birth': date_of_birth,
        'sex': sex,
        'email': email,
        'atsi': atsi_status,
        'password': password,
        'username': username,
     }

    return user_data

```


### Put the generated fake user profiles into a Pandas DataFrame.

We can use the function we created above to generate a list of fake user profiles. We can then use the Pandas DataFrame class to create a DataFrame from the list of user profiles. This will allow us to easily manipulate the data and export it to a CSV file. I'll also use the `tabulate` package to make the output look nice.

```python
users = []
for i in range(10):
    users.append(create_user_australian(random.choice(['M', 'F'])))

df = pd.DataFrame(users)

print(tabulate(df, headers='keys', tablefmt='simple_outline'))
```


And the results should look something like this:

[![User profiles]({{ site.url }}{{ site.baseurl }}/assets/images/user_profiles_sml.jpg)]({{ site.url }}{{ site.baseurl }}/assets/images/user_profiles.jpg)


### Conclusion

The Python Faker package is a powerful tool for generating fake user data quickly and efficiently. It provides a wide range of providers and methods to generate various types of data such as names, addresses, email addresses, and phone numbers. By customizing the generated data according to your needs, you can easily create realistic-looking user profiles for testing or development purposes.