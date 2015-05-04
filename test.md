Integrating with Nationbuilder to do a manual user login
==============

### Introduction
This tutorial will explain how to integrate Django with Nationbuilder to login a registered user in a Nationbuilder database.  The code covered in this tutorial will take in user login info entered on a Django page and compare it to records in a Nationbuilder database. It will check for a succesfull data match to authenticate the login or return an error specifying the reason why the login was not successful.

### Nationbuilder
Nationbuilder is a software platform that allows nonprofits political parties to organize their communities.  Nationbuilder handles a people database, website, communications, and donations for parties using it.  Our customer's system was based on Nationbuilder and for our project we interfaced with the information stored there.

### Sending form data
This tutorial will assume that a simple Django page with a form is being used to login. All that is being used in this example is a form with a field for the user to enter in their email address.
In views.py, start by sending the form login info as POST data:
```
email_form = EmailForm(request.POST)
if email_form.is_valid():
  email = form.cleaned_data['email']
```
This simply checks whether valid form data is being sent as POST data and normalizes it to keep the format consistent.

### Making the API call
Next we will make an API call to the Nationbuilder databse to search for a user whose email matches the one that is entered into the form (note that nation_slug should correspond to your Nationbuilder sites' slug):
```
try:
	            response = session.get("https://"+nation_slug+".nationbuilder.com/api/v1/people/match?email="+volunteerId,
					params={'format': 'json'},
					headers={'content-type': 'application/json'})
	            jsonResponse = json.loads(response.text)
	            if jsonResponse["person"] is not None:
	            	id = jsonResponse["person"]["id"]
          
```
By setting the params field to 'json' we indicate that the return string will be in JSON. We can then use the json.loads() function to convert the returned JSON string to a nested Python dictionary.
If a person exists in the Nationbuilder database that matches the email entered, we save their id listed in the database.

### Error checking
There are 3 things we need to check for errors so in our code: Whether the if statement could find a matching email, whether the API call succeeded, and whether the email form data was vaild.
Add these lines to check for these 3 possibilities:
```
else:
	            	return render(request, 'theHaven/register.html', {'email_form': email_form, 'given_email': email, 'success': "email_fail"})
            except ValueError:
        		return render(request, 'theHaven/register.html', {'email_form': email_form, 'given_email': email, 'success': "api_fail"})

     else:
         email_form = EmailForm()
         return render(request, 'theHaven/register.html', {'email_form': email_form})
```
The first else statement checks whether the email was not found. The except statement checks whether the API call failed, and the 2nd else statement checked for valid form data.
If any of these fails, the render() function renders our template again, passing in appropriate variables that correspond to why the error occurred.

### Final Code
```
email_form = EmailForm(request.POST)
         if email_form.is_valid():
            email = form.cleaned_data['email']
            try:
	            response = session.get("https://"+nation_slug+".nationbuilder.com/api/v1/people/match?email="+volunteerId,
					      params={'format': 'json'},
					      headers={'content-type': 'application/json'})
	            jsonResponse = json.loads(response.text)
	            if jsonResponse["person"] is not None:
	            	id = jsonResponse["person"]["id"]
	            else:
	            	return render(request, 'theHaven/register.html', {'email_form': email_form, 'given_email': email, 'success': "email_fail"})
           except ValueError:
        		return render(request, 'theHaven/register.html', {'email_form': email_form, 'given_email': email, 'success': "api_fail"})

     else:
         email_form = EmailForm()
         return render(request, 'theHaven/register.html', {'email_form': email_form})
```

### Conclusion
Thank you for using this tutorial! You can now make a simple Nationbuilder authentication page that passes error checking variables to a template.



