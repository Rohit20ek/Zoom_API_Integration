# Zoom_API_Integration
Automating the zoom with the google sheet to reduce the efforts in downloading and exporting it at a one place.it will automatically fetch the data and put it to the google worksheet 


Challenge
This is also the case for my company and my Church’s bible study organisation. In the case of my bible study activities, we usually would take attendance. When it was done in person, it was easy as we were broken up to a few classes. But, since we moved to Zoom, a few classes and sometimes, all of them are combined in one Zoom bible study session.
This presented a challenge in terms of taking attendance as people might join and leave at different times, requiring the class leaders to keep monitoring the list of participants. It is really difficult to do and it would mean that the class leaders are probably not going to be able to pay attention to the session, itself.


Solution
Now, since I am a Dev, myself, I took the liberty of looking into Zoom API and thought I could automate it all. So the class leaders do not have to manually take the attendance, but rather just wait for my report to be generated after each of the bible study classes.
The Zoom API is quite simple and straight forward to implement. The API that I use is the report meeting participants API. It does require a Zoom pro account, but fortunately, my organisation decided to subscribe to it as we are going to be using it for a while. So, no dramas there. 😃
I chose Python as the programming language to use as it is very convenient and definitely made for this type of task. So first, my Python program will call the Zoom API to fetch the list of meeting participants. Then, using Pandas library, it will transform and simplify the data to a human friendly, readable tabular format. Finally, I am using the Google API to store the final report in a Google Sheets, which is made accessible to the class leaders who need to see the attendance report.

So far the feedback has been positive. It solves the challenge I mentioned above. I thought that if it is useful for us, it might be useful for you and your organisations too. That is why I am sharing the detailed step-by-step solution with you below.

Zoom API Authentication
The first step to use the Zoom API is the authentication. They offer two methods of authentication, which are OAuth 2.0 and JWT. I want to have the ability to use the API without first obtaining a permission from a resource owner (a Zoom user). Also, it needs to be simple and quick to implement. This leaves the JWT method as the winner.
Zoom actually also recommends JWT for the purpose of using their API if it’s only going to be used by users that are in the Zoom account. As I said, this is only for reporting purpose so it fits this scenario well. This is what the official doc says.


If your app is meant to be used only by yourself or by users that are in your Zoom account, it is recommended that you use JWT for authentication.
The JWT method allows us to do server-to-server authentication. This is because in the signed JWT payload we send to Zoom, it contains our API Key and Secret. This JWT payload needs to be sent in every request to Zoom in the Authorization header as a Bearer token.

If your app is meant to be used only by yourself or by users that are in your Zoom account, it is recommended that you use JWT for authentication.
The JWT method allows us to do server-to-server authentication. This is because in the signed JWT payload we send to Zoom, it contains our API Key and Secret. This JWT payload needs to be sent in every request to Zoom in the Authorization header as a Bearer token.

Then, select the JWT option as your app type. Notice that because my account already has a JWT app registered prior to writing this tutorial, Zoom notifies me of that by prompting with “Your account already has JWT credentials”. Also, it does not have the blue Create button.
If you have never created a JWT app before, your JWT option should have the blue Create button. So, go ahead and click on the blue Create button.

Fill in App information
That will bring you to the App information page that looks like this. Go ahead and fill in the information there. The next step after you finish with this one is to generate the app credentials.

Generate credentials
Now, after you progress from the app information section, you will see your app credentials page that looks like this. Once you’ve reached this step, you are pretty much ready to interact with the Zoom API. 

Notice there’s an option to View JWT Token at the bottom left corner. This will actually generate a JWT token for you and using it, you can make an API call to Zoom. You can choose the expiry period as well.

Google API Authentication
Just like with Zoom API, we also need to authenticate ourselves (or our app) in order to be able to use Google API. We are going to use their Drive API and Sheets API.
The Drive API is to get a folder location in which we want to store the Google Sheets containing the Zoom meeting attendance report. Again, I do not want to have to require a resource owner permission for this. So, I am going to use the Google OAuth 2.0 with a service account as the authentication method.
A service account essentially is an account that belongs to our application, not to our individual user account. Our application then calls the Google API on behalf of the service account.

Create a new project
First, let’s create a new project in the Google Console Developer dashboard. Click on the drop down arrow on the top left hand corner next to Google APIs writing. In my screenshot below, it’s the one that says First Medium Project.

It will pop up a prompt window and on the top right hand corner, there’s the New Project option. Go ahead and click on New Project.

You can then specify the name of your project. Here, I am just going to use what Google generated for me. Look at the Project ID. It’s pretty cool, isn’t it? 😆 Once you’re done, click on the blue Create button.

You will be taken to the Dashboard page once the project is initialised.

Enable Google Drive and Sheets API
After our project is ready, let’s enable the APIs that we want to use. In this case, they are the Google Drive and Sheets APIs.
Step 1. Let’s click on the Enable APIs and Services button.
Step 2. On the search bar, type in “Drive” and it should give you the list of available APIs that match your search query. Click on the Google Drive API.
Step 3. Finally, click on the blue Enable button to enable this API in our project.
Now, go back to the Dashboard by clicking on the hamburger icon on the top left hand corner as shown in the picture below.
Repeat the above three steps but this time, it is for the Google Sheets API. So, for step 2, you should type in “Sheet” on the search bar.
Set up credentials
After you have enabled the Google Sheets API, go to the Credentials page of your project as shown in below picture.
Then, click on the Create Credentials option and choose Service account.
It will bring you to a page where you can set the name of your service account and give a description of what it does. Once you are finished, click on the Create button.
Skip the second step by clicking on the Continue button.
In the last step, we need to create a key as this is required for the server-to-server authentication. Click on the Create Key button.
Choose the JSON option as the key type and click on Create. This will prompt you to download a JSON file. Store it in your project folder or somewhere safe in your computer.

Once the key is saved, click on the Done button to finish our credentials setup. We are now done with Google API authentication.
In the next section, we will jump into writing the code, which I know you all have been waiting for. 

Python Program
This is the most fun part of this article. Congratulations on arriving here. 🐵
The Python program is simple. It will have two classes, one to interact with Zoom API and the other to interact with Google API. It will have a main Python file which is the entry point of the program.
I assume you have knowledge in setting up a Python virtual environment. We are going to need to use it for this project as to isolate the dependencies from your global Python environment. You can read about it in this tutorial.
Note that the Python version I am using for this project is Python 3.8.

Requirements
Let’s create requirements.txt file to contain all the dependencies we need for this project.

requests==2.23.0
Authlib==0.14.1
pandas==1.0.3
google-api-python-client==1.8.0


Let’s go through why we need each of the libraries above.
requests is required as our program needs to make a HTTP request to Zoom API endpoints. At the time of writing, I am not aware of any Python Zoom client library.
Authlib is needed for JWT token creation. Remember, the Zoom API requires our program to send JWT token in every request placed in the Authorization header as a Bearer token.
pandas is used to transform the response from Zoom API into a nice, human readable table.
google-api-python-client is the official Python client library for Google API. This client library makes it easy for us to interact with Google API.
Zoom class
We are going to define a class that defines the properties and methods of our Zoom object. Remember that Python is an object oriented programming. A class is just a template or blueprint that we use to construct or initialise an object.
In our Zoom class definition, we are going to have 2 methods, one for JWT creation and the other for calling the report meeting participants API. Let’s go ahead and create a new Python file named zoom.py.

The class arguments api_key and api_secret will contain the values of our Zoom API key and API secret we generated in the Zoom API Authentication section above.
We also define the expiration time for our JWT token, which is 30 minutes. And the algorithm we are using to sign our JWT token is HS256. As discussed, we are passing this token in the Authorization header as a Bearer token as can be seen in the get_meeting_participants method.
Notice that for the get_meeting_participants method, we have a parameter called next_page_token which is an Optional type. This is because the Zoom API we call will return a response that can contain only a maximum of 300 participants per page.
The next_page_token is passed in the case where we have over 300 participants for the particular meeting we are extracting the data for.

Google class
Just like our Zoom class, we also need a class that will handle our interaction with Google API. To avoid naming conflict with the classes contained in the Python client libraries, we will name our Google class as Googl instead. Now, go ahead and create a new file named googl.py.

Our Googl class will take two arguments. The service_account_file argument is the path to the JSON file we downloaded into our computer after we created the service account. As a suggestion, you can create a folder named .secrets inside your root project folder and store the file there. Also put this in your .gitignore file to avoid accidentally committing it to your Git repository.
The scope argument will contain the list of Google OAuth 2.0 scopes that we want our program to have. In our case, we are going to need two scopes:
To authorise our program, we use the service account method and you can see how we generate the credentials in this line of code. It requires both the service account file and the scope needed.

self.creds = service_account.Credentials.from_service_account_file(self.service_account_file,                                                                     scopes=self.scopes)

These two lines of code construct the Google Drive and Google Sheet resources that we need to interact with those two Google APIs.

self.drive = discovery.build("drive", "v3", credentials=self.creds)        self.sheets = discovery.build("sheets", "v4", credentials=self.creds)

The first method get_folder_id is used to get the folder id in our Google Drive, which our Google Sheets containing Zoom reports will be stored in. This folder id is known as the parent folder id, which is a parameter to the method named create_new_sheet.
The create_new_sheet method does the creation of a new and empty Google Sheet in a folder specified by its parent_folder_id parameter. This new Sheet will have a name specified by its file_name parameter. This method returns the id of the Sheet that just got created. We need this id later when our program is inserting the report to the Google Sheet so it knows which Sheet to insert it into. This is what the insert_df_to_sheet is responsible for.
The insert_df_to_sheet method will insert values as raw values starting from cell A1. Before inserting the values from the DataFrame, we also get rid of the index as done in this line of code.

"values": df.T.reset_index().T.values.tolist()

Lastly, the get_sheet_link method is used to get the link to a specific Google Sheet as specified by the google_sheet_id parameter. The purpose of this is so that we can send the link to the consumers of the file and they can go to the file directly without having to navigate through the Google Drive.

Main file
It’s time to write the entry point to our program, which is the main.py file. This file is like the orchestrator, it will construct both the Zoom and Googl objects and use them in a scripting fashion to get the result that we want, which is a table in a Google Sheet. It also uses pandas to construct the DataFrame before inserting to the Google Sheet.

Let’s see how the main.py looks like.


The first part of the script is about assigning environment variables and other constants that are needed by our Zoom and Googl objects. Nothing complicated here.

ZOOM_API_KEY = os.environ.get("ZOOM_API_KEY")
ZOOM_API_SECRET = os.environ.get("ZOOM_API_SECRET")
ZOOM_MEETING_ID = os.environ.get("ZOOM_MEETING_ID")
SERVICE_ACCOUNT_FILE = f".secrets/{os.listdir('.secrets')[0]}"
SCOPES = ["https://www.googleapis.com/auth/drive", "https://www.googleapis.com/auth/drive.file"]

Next, we initialise our Zoom object and call the report meeting participants API with the JWT token. We store the participants contained in the response in the list_of_participants variable. Then, we have a while loop that checks for the next_page_token and if it exists, the program will call the Zoom API again and add the result to the list_of_participants variable until there’s no more next_page_token.

zoom = Zoom(ZOOM_API_KEY, ZOOM_API_SECRET)

jwt_token: bytes = zoom.generate_jwt_token()
response: Response = zoom.get_meeting_participants(ZOOM_MEETING_ID, jwt_token)
list_of_participants: List[dict] = response.json().get("participants")

while token := response.json().get("next_page_token"):
    response = zoom.get_meeting_participants(ZOOM_MEETING_ID, jwt_token, token)
    list_of_participants += response.json().get("participants")
    
    We convert the list_of_participants variable into a DataFrame, dropping the attentiveness_score column as I think that it is not a reliable measure (sorry, Zoom 😋). For my case, I convert the join_time and leave_time into AEST timezone. I also sort the table by id, name and join_time to make it easy to read and follow.
You might notice that besides id, we also have user_id. Based on my observation on the data, user_id does not seem to be unique. In the case where a participant rejoins a few times, each of his or her entries will have a different user_id but always the same id. The id seems to be the identifier of one’s Zoom account and if one doesn’t have a Zoom account, he or she will not have the id value.

df: DataFrame = pd.DataFrame(list_of_participants).drop(columns=["attentiveness_score"])
df.join_time = pd.to_datetime(df.join_time).dt.tz_convert("Australia/Sydney")
df.leave_time = pd.to_datetime(df.leave_time).dt.tz_convert("Australia/Sydney")

df.sort_values(["id", "name", "join_time"], inplace=True)

Now, for my bible study group, we would like to know how long each participant is staying in the Zoom session. To do this, I need to do some aggregation to get the total duration of each participant from their earliest join_time and latest leave_time.

Why earliest and latest? Because, I notice that sometimes people’s connection might drop and they log back in. Zoom records this as two separate entries.

output_df: DataFrame = df.groupby(["id", "name", "user_email"]) \
    .agg({"duration": ["sum"], "join_time": ["min"], "leave_time": ["max"]}) \
    .reset_index() \
    .rename(columns={"duration": "total_duration"})
    
    After the aggregation, we need to rename the columns. This is because the aggregation creates a multilevel index like so.
    
    >>> output_df.columns
MultiIndex([(            'id',    ''),
            (          'name',    ''),
            (    'user_email',    ''),
            ('total_duration', 'sum'),
            (     'join_time', 'min'),
            (    'leave_time', 'max')],
           )
           
           To do this, we need to re-assign the columns property of our output_df data frame to the index of level 0.
           
           output_df.columns = output_df.columns.get_level_values(0)
           
           
           After we’ve done this, the column names will be the ones before the aggregation plus the new column total_duration with no other index.
           
           >>> output_df.columns
Index(['id', 'name', 'user_email', 'total_duration', 'join_time',
       'leave_time'],
      dtype='object')
      
      Note that the total_duration as of this point is in seconds and for my bible study activity, it usually goes up to over 2 hours. So, to make it easy for the consumers of the report to read, I am going to convert from seconds to hours.
      
      output_df.total_duration = round(output_df.total_duration / 3600, 2)
      
      Furthermore, to make it easier to read, I am formatting the join_time and leave_time by dropping the UTC timezone offset.
      output_df.join_time = output_df.join_time.dt.strftime("%Y-%m-%d %H:%M:%S")
output_df.leave_time = output_df.leave_time.dt.strftime("%Y-%m-%d %H:%M:%S")

Next, I want the name of the Google Sheet to contain the date of the meeting.
meeting_date: str = output_df.join_time.tolist()[0].split(" ")[0]
output_file: str = f"zoom_report_{meeting_date}"

It’s time to do the operations with Google API so we will initialise the Googl object, passing the SERVICE_ACCOUNT_FILE and SCOPES variables to the class constructor.

googl = Googl(SERVICE_ACCOUNT_FILE, SCOPES)
We are going to get the folder id where we want to keep the Google Sheets in. To do this, we will use the get_folder_id method from our Googl class. In my case, the name of the folder is Zoom, which is passed as an argument to the method.
For this method to succeed, the folder with the name you pass as an argument to the method has to exist or else this method will return an error IndexError: list index out of range. Feel free to enhance this method to handle such case 🙂.

zoom_folder_id: str = googl.get_folder_id("Zoom")
Then, we will create a new Google Sheet in the Zoom folder. This is done via the create_new_sheet method. Remember that this method returns the Google Sheet id.
sheet_id = googl.create_new_sheet(output_file, zoom_folder_id)

We are now ready to insert the values of our output_df to the Google Sheet we just created. We will call the insert_df_to_sheet method to get this task done. By passing the sheet_id, we can tell Google which Google Sheet we want the values to be inserted.
result = googl.insert_df_to_sheet(sheet_id, output_df)

Finally, we will get the link to that Google Sheet so we can share it with the consumers of the report.

sheet_link = googl.get_sheet_link(result.get("spreadsheetId"))
For my case, I am printing key information to the console just for sense checking purposes.

print(f"Finished uploading Zoom report.\n"
      f"spreadsheetId: {result.get('updates').get('spreadsheetId')}\n"
      f"updatedRange: {result.get('updates').get('updatedRange')}\n"
      f"updatedRows: {result.get('updates').get('updatedRows')}\n"
      f"link: {sheet_link}")
      
      
That’s it. By using this script, we have automated the Zoom meeting attendance report generation.
Setting environment variables
In order to run the main.py, we need to set up a few environment variables: ZOOM_API_KEY, ZOOM_API_SECRET and ZOOM_MEETING_ID. The way I do it is by having a shell script called setup.sh, which I will execute once before I run the main.py script.
The content of the setup.sh script looks like this.
#!/bin/sh
export ZOOM_API_KEY=yourZoomApiKeyHere
export ZOOM_API_SECRET=yourZoomApiSecretHere
export ZOOM_MEETING_ID=yourZoomMeetingIdHere
Each time before running the main.py, you can run the above script from terminal like so.

$ . ./setup.sh
Run the main script
To run the main.py script, go to your terminal and execute the following commands.
$ . ./setup.sh
$ source venv/bin/activate
$ python main.py
Project folder structure
The final project folder structure looks like this.
zoom-reporting
|_ .gitignore
|_ .secrets
    |_ solid-heaven-1a59mo2lq384.json
|_ venv
|_ googl.py
|_ main.py
|_ requirements.txt
|_ setup.sh
|_ zoom.py
You can visit this Github repo for the finished code. Remember, the repo does not contain the .secrets folder and the JSON file. You need to create this on your local computer.
Also, remember to edit the setup.sh to include your Zoom credentials.
Furthermore, you also need to set up the Python virtual environment and install the dependencies listed in the requirements.txt file before running the program.

Wrap Up
I am sure you have thought of a few improvements you can make to the existing project. One example would be to upload this function to AWS lambda and set a cron job to run it after the meeting finishes given that your meeting always occurs at the same time.
Zoom also provides a functionality: event, which means you can set up a Webhook for Zoom to send a request notifying you of events of interests, e.g. a meeting has finished. This means you can have another lambda function that listens to that event and then, call the other lambda function that does the Zoom report generation.
Anyways, the possibilities are endless. So, go use your creativity and build a more robust solution than this one. 
