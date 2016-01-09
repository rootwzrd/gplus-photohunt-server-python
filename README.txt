
PhotoHunt: Python on Google App Engine

This variation of PhotoHunt runs as a Python application on Google App Engine. The app demonstrates how to build a social application that uses Google+ Sign-In, personalization, app activities, over-the-air install, and interactive posts.

Google App Engine allows rapid development and deployment of applications and allows your application to scale to a very significant capacity. If you use a different application server, don't worry, this guide explains the important pieces to help understand how to integrate with your existing Python app.

This guide discusses in great detail the aspects of PhotoHunt that relate to the Google+ Platform; however, parts of PhotoHunt that are not directly related to Google+ are not discussed. For more information on how PhotoHunt handles the code that is not discussed, such as uploading photos, see the Google App Engine documentation or the AngularJS documentation.

Browsing the source code

If you would like to browse the source code before digging in too deeply, simply have a look at PhotoHunt's Python repository in GitHub.

To ease the set up of the PhotoHunt app, the Git project contains a version of the Google APIs Client Library for Python. When developing your own app, you should download the latest version of the library.

Requirements

PhotoHunt Python has the following requirements:

Python 2.7
Setting up PhotoHunt

PhotoHunt Python can be set up in three steps: set up App Engine, create a Google Developers Console project, and apply the settings from these steps to the source code of PhotoHunt Python.

Step 1: Create a Google App Engine project

You can use the Google App Engine SDK to execute the sample application on a local development server.

Navigate to the Google App Engine Admin Console .
Click Create Application.
Choose an application identifier for your app:
Type an unique Application Identifier into the text box.
Verify that the identifier is available by clicking Check Availability. If not available, try a new identifier.
Note or copy the Application Identifier for use in a later step.
Click Create Application.
Download and install the Google App Engine SDK for Python . You can skip this step if the Google App Engine SDK for Python is installed on your system.
Step 2: Enable the Google+ API

You need to enable the Google+ API for your app. You can do this by creating a project for your app in the Google Developers Console.

To create a Developers Console project, create an OAuth 2.0 client ID, and register your Web origins and redirect URIs:

Go to the Google Developers Console  .
Select a project, or create a new one by clicking Create Project:
Note: Use a single project to hold all platform instances of your app (Android, iOS, web, etc.), each with a different Client ID.

In the Project name field, type in a name for your project.
In the Project ID field, optionally type in a project ID for your project or use the one that the console has created for you. This ID must be unique world-wide.
Click the Create button and wait for the project to be created.
Click on the new project name in the list to start editing the project.
In the left sidebar, select the APIs item below "APIs & auth". A list of Google web services appears.
Find the Google+ API service and set its status to ON—notice that this action moves the service to the top of the list. You can turn off the Google Cloud services if you don't need them. Enable any other APIs that your app requires.
In the sidebar under "APIs & auth", select Consent screen.
Choose an Email Address and specify a Product Name.
In the sidebar under "APIs & auth", select Credentials.
Click Create a new Client ID — a dialog box appears. 
Register the origins from which your app is allowed to access the Google APIs, as follows. An origin is a unique combination of protocol, hostname, and port.
In the Application type section of the dialog, select Web application.
In the Authorized JavaScript origins field, enter the origin for your app. You can enter multiple origins to allow for your app to run on different protocols, domains, or subdomains. Wildcards are not allowed. In the example below, the second URL could be a production URL.
http://localhost:8080
https://myproductionurl.example.com
In the Authorized redirect URI field, delete the default value. Redirect URIs are not used with JavaScript APIs.
Click the Create Client ID button.
In the resulting Client ID for web application section, copy the Client ID and Client secret that your app will need to use to access the APIs.
Step 3: Set up the application

Get the latest version of PhotoHunt Python. One way is to use git to clone the application repository.

git clone https://github.com/googleplus/gplus-photohunt-server-python.git
This command creates a gplus-photohunt-server-python directory in your current working directory. You can also download and extract a ZIP file of the project.

Edit the app.yaml file, insert your Google App Engine application identifier that you created in step 1 in the application: parameter, and save the file.
In static/js/services.js, replace the value of YOUR_CLIENT_ID with the client ID that you generated in step 2 in the Developers Console.
In client_secrets.json, replace the values of YOUR_CLIENT_ID and YOUR_CLIENT_SECRET with the client ID and client secret that you generated in step 2.
Run the project by executing the following command on the command line from the directory above your project:

/path/to/appengine/sdk/dev_appserver.py <project directory>
Navigate to http://localhost:8080 to try the app.
Note: To use additional clients, and enable app activities and interactive posts, you must deploy the PhotoHunt server to Google App Engine.
Application layout

PhotoHunt Python is designed with a common MVC pattern. The model is represented by the classes that are defined in the model.py module, the controller is represented by the webapp2 RequestHandlers that are defined in the handlers.py module, and the views are represented by the resources within the static directory, which includes HTML files, images, JavaScript, and CSS stylesheets.

All of the libraries that are required by the Python source code reside in the root directory of the project. The Python module files are bundled to ease the set up process for the application, and to get you working as quickly as possible. If you have additional Python dependencies, put their files in the project root directory and they will be uploaded with the project.

The mapping of URLs to handlers is done at the end of handlers.py. Edit this file if you add more handlers or you want to change URLs.

The Google App Engine project configuration is set in app.yaml. Edit this file to configure your project settings such as your application's version.

The front-end code for all of the PhotoHunt languages makes use of AngularJS and the Google API JavaScript Client. Both of these items are included and used in multiple places throughout static/index.html and static/js.

Writing PhotoHunt, feature by feature

When users interact with the PhotoHunt web app, there are three situations for how users might discover and use the app:

A user discovers the app on their own.
A friend invites them to the app by using interactive posts.
A pre-existing user of the app finds their friends from Google+ are also using the app.
The following sections explain these situations using the fictional characters: Alice, Bob, Charles, and David.

Introducing Alice, PhotoHunt's newest user

Alice represents a brand new user who discovers PhotoHunt for the first time by doing a web search or a search in the Google Play Store or Apple App Store.

Signs in with Google.
Uploads a photo.
Promotes the photo to Bob by asking him to vote.
Invites Charles to join PhotoHunt.
Views David's photos.
Disconnects from PhotoHunt.
Introducing Alice's friends

Bob

Bob is a friend of Alice, but he's never used PhotoHunt. He received Alice's interactive post notification. He enjoys Alice's photos and wants to vote for her photo.

Sees notification of share from Alice.
Clicks the Vote button in the post.
Signs in with Google.
Charles

Charles is another friend of Alice, and also has never used PhotoHunt.

Sees notification of share from Alice.
Clicks the Join button in the post.
Signs in with Google.
David

David is a friend of Alice who has used PhotoHunt for a long time prior to Alice joining. Alice and David have each other in their circles on Google+.

Alice has already uploaded photos.
David signs in with Google.
David uploads photos.
David and Alice see each other's photos prominently.
PhotoHunt's code for Alice

The implementation to provide Alice's flow through PhotoHunt uses the Google+ Sign-In button, over-the-air install, app personalization, interactive posts, and app activities.

Next, we'll walk through how the app integrates the Google+ platform to provide these features to Alice.

Google+ Sign-In button

Visitors who are not signed in see the Google+ Sign-In button in the top-right of the page. Clicking the button prompts the user to authorize the application if the user is not already connected with the app. They see an OAuth 2.0 permissions dialog that lists the information and services that the app is requesting to access on behalf of the user.

First, an HTML element is added to static/index.html that represents the sign-in button. PhotoHunt uses the standard Google+ Sign-In button rather than rendering a custom button.

<span id="signin" ng-show="immediateFailed">
  <span id="myGsignin"></span>
</span>
In the code above, the important piece is that the inner span has an ID of myGsignin, which is used in the JavaScript in static/js/controllers.js.

The following code has the simple purpose of calling gapi.signin.render(), and providing the correct callback to use when the user is actually signed in:

$scope.signIn = function(authResult) {
  $scope.$apply(function() {
    $scope.processAuth(authResult);
  });
}

$scope.processAuth = function(authResult) {
  $scope.immediateFailed = true;
  if ($scope.isSignedIn) {
    return 0;
  }
  if (authResult['access_token']) {
    $scope.immediateFailed = false;
    // Successfully authorized, create session
    PhotoHuntApi.signIn(authResult).then(function(response) {
      $scope.signedIn(response.data);
    });
  } else if (authResult['error']) {
    if (authResult['error'] == 'immediate_failed') {
      $scope.immediateFailed = true;
    } else {
      console.log('Error:' + authResult['error']);
    }
  }
}

$scope.renderSignIn = function() {
  gapi.signin.render('myGsignin', {
    'callback': $scope.signIn,
    'clientid': Conf.clientId,
    'requestvisibleactions': Conf.requestvisibleactions,
    'scope': Conf.scopes,
    'apppackagename': 'your.photohunt.android.package.name',
    'theme': 'dark',
    'cookiepolicy': Conf.cookiepolicy,
    'accesstype': 'offline'
  });
}
After a user is successfully signed in, a call is made to http://localhost:8080/api/connect to connect the user to the PhotoHunt server, and to retrieve and store some information about the user. This step is done in ConnectHandler, within the post() method:

def post(self):
  """Exposed as `POST /api/connect`.

  Takes the following payload in the request body.  The payload represents
  all of the parameters that are required to authorize and connect the user
  to the app.
  {
    'state':'',
    'access_token':'',
    'token_type':'',
    'expires_in':'',
    'code':'',
    'id_token':'',
    'authuser':'',
    'session_state':'',
    'prompt':'',
    'client_id':'',
    'scope':'',
    'g_user_cookie_policy':'',
    'cookie_policy':'',
    'issued_at':'',
    'expires_at':'',
    'g-oauth-window':''
  }

  Returns the following JSON response representing the User that was
  connected:
  {
    'id':0,
    'googleUserId':'',
    'googleDisplayName':'',
    'googlePublicProfileUrl':'',
    'googlePublicProfilePhotoUrl':'',
    'googleExpiresAt':0
  }

  Issues the following errors along with corresponding HTTP response codes:
  401: The error from the Google token verification end point.
  500: 'Failed to upgrade the authorization code.' This can happen during
       OAuth v2 code exchange flows.
  500: 'Failed to read token data from Google.'
       This response also sends the error from the token verification
       response concatenated to the error message.
  500: 'Failed to query the Google+ API: '
       This error also includes the error from the client library
       concatenated to the error response.
  500: 'IOException occurred.' The IOException could happen when any
       IO-related errors occur such as network connectivity loss or local
       file-related errors.
  """
  # Only connect a user that is not already connected.
  if self.session.get(self.CURRENT_USER_SESSION_KEY) is not None:
    user = self.get_user_from_session()
    self.send_success(user)
    return

  credentials = None
  try:
    connect_credentials = json.loads(self.request.body)
    if 'error' in connect_credentials:
      self.send_error(401, connect_credientials.error)
      return
    if connect_credentials.get('code'):
      credentials = ConnectHandler.exchange_code(
          connect_credentials.get('code'))
    elif connect_credentials.get('access_token'):
      credentials = ConnectHandler.create_credentials(connect_credentials)
  except FlowExchangeError:
    self.send_error(401, 'Failed to exchange the authorization code.')
    return

  # Check that the token is valid and gather some other information.
  token_info = ConnectHandler.get_token_info(credentials)
  if token_info.status_code != 200:
    self.send_error(401, 'Failed to validate access token.')
    return
  logging.debug("TokenInfo: " + token_info.content)
  token_info = json.loads(token_info.content)
  # If there was an error in the token info, abort.
  if token_info.get('error') is not None:
    self.send_error(401, token_info.get('error'))
    return
  # Make sure the token we got is for our app.
  expr = re.compile("(\d*)(.*).apps.googleusercontent.com")
  issued_to_match = expr.match(token_info.get('issued_to'))
  local_id_match = expr.match(CLIENT_ID)
  if (not issued_to_match
      or not local_id_match
      or issued_to_match.group(1) != local_id_match.group(1)):
    self.send_error(401, "Token's client ID does not match app's.")
    return

  # Store our credentials with in the datastore with our user.
  user = ConnectHandler.save_token_for_user(token_info.get('user_id'),
                                            credentials)

  # Fetch the friends for this user from Google, and save them.
  ConnectHandler.get_and_store_friends(user)

  # Store the user ID in the session for later use.
  self.session[self.CURRENT_USER_SESSION_KEY] = token_info.get('user_id')

  self.send_success(user)
As the comments for the method state, the post() method takes the token data given by the Google+ Sign-In button callback, and upgrades the attached authorization code into a fully-qualified access token and refresh token pair. This token data is specific to Alice. Then, this token pair is stored along with the User object that represents Alice in our datastore. If your app has an existing Facebook or Twitter sign-in integration, this is the point at which you would create the relevant new fields that are mentioned in model.User, including:

google_user_id
google_display_name
google_public_profile_url
google_public_profile_photo_url
google_access_token
google_refresh_token
google_expires_in
google_expires_at
post() has only set the google_user_id, google_access_token, google_refresh_token, google_expires_in, and google_expires_at fields for Alice. The display name, public profile URL, and public profile photo URL will be set later when the app uses the Google+ APIs to personalize PhotoHunt for Alice.

After the token is stored, the app sets Alice's ID in her session so that the app can use the ID in future requests. The manner in which you would do cross-request user identification in a real application varies, but this is one way.

Finally, post() fetches a list of the people that Alice chose to share with PhotoHunt from her Google+ circles. Fetching a user's social graph is discussed below in app personalization.

Over-the-air install

With one line of code, you can enable the over-the-air install feature to prompt your Android users to install your Android app when they sign in to your web app. The following example highlights this parameter:

$scope.renderSignIn = function() {
  gapi.signin.render('myGsignin', {
    // ...
    'apppackagename': 'your.photohunt.android.package.name',
    // ...
  });
}
Be sure to replace your.photohunt.android.package.name with the package name for your own Android app.

After adding the apppackagename parameter, the over-the-air install happens automatically if certain conditions are met:

The client IDs for the web app and Android are created in the same Developers Console project.
The app meets the quality threshold requirement and the app is published in the Google Play store.
The user does not already have your app installed and it is compatible with at least one of their Android devices.
App personalization

PhotoHunt is personalized for Alice in two ways:

Her Google+ name, profile photo, and profile link are shown in the PhotoHunt interface.
She automatically sees photos from the people that she chose to share with PhotoHunt, who have also chosen to share photos with Alice.
Alice's name, profile photo, and profile link are retrieved when the /api/connect endpoint is called. This is done as part of the save_token_for_user() method.

def get_user_profile(credentials):
  """Return the public Google+ profile data for the given user."""
  http = httplib2.Http()
  plus = build('plus', 'v1', http=http)
  credentials.authorize(http)
  return plus.people().get(userId='me').execute()

def save_token_for_user(google_user_id, credentials):
  """Creates a user for the given ID and credential or updates the existing
  user with the existing credential.

  Args:
    google_user_id: Google user ID to update.
    credentials: Credential to set for the user.

  Returns:
    Updated User.
  """
  user = model.User.all().filter('google_user_id =', google_user_id).get()
  if user is None:
    # Couldn't find User in datastore.  Register a new user.
    profile = ConnectHandler.get_user_profile(credentials)
    user = model.User()
    user.google_user_id = profile.get('id')
    user.google_display_name = profile.get('displayName')
    user.google_public_profile_url = profile.get('url')
    image = profile.get('image')
    if image is not None:
      user.google_public_profile_photo_url = image.get('url')
  user.google_credentials = credentials
  user.put()
  return user
The relevant lines are those within the if user is None: block. A Google API Python Client instance is created for the Google+ API, and then a call is made to the people.get endpoint of the Google+ API within get_user_profile(), with a user ID of me. In this case, because the client is authorized for Alice, me tells Google+ to return Alice's public profile information.

The last step of the call to /api/connect is to fetch a list of the people that Alice chose to share with PhotoHunt from her Google+ circles. This is done by the get_and_store_friends() method.

def get_and_store_friends(user):
  """Query Google for the list of the user's friends that they've shared with
  our app, and then store those friends for later use.

  Args:
    user: User to get friends for.
  """

  # Delete the friends for the given user first.
  edges = model.DirectedUserToUserEdge.all().filter(
      'owner_user_id = ', user.key().id()).run()
  db.delete(edges)

  http = httplib2.Http()
  plus = build('plus', 'v1', http=http)
  user.google_credentials.authorize(http)
  friends = plus.people().list(userId='me', collection='visible').execute()
  for google_friend in friends.get('items'):
    # Determine if the friend from Google is a user of our app
    friend = model.User.all().filter('google_user_id = ',
        google_friend.get('id')).get()
    # Only store edges for friends who are users of our app
    if friend is not None:
      edge = model.DirectedUserToUserEdge()
      edge.owner_user_id = user.key().id()
      edge.friend_user_id = friend.key().id()
      edge.put()
This method is meant to highlight how to retrieve a list of people, but not how to do it efficiently for your own application. This method can take a few seconds to run if the user has a large number of people they've chosen to share with PhotoHunt. Because of this issue, we recommend that you fork this process into a thread, task queue, cron job, or some other asynchronous execution mechanism.

The method creates another Google API Python Client instance for Google+, and calls the people.list endpoint while authorized as Alice. This method returns all of the people that Alice has chosen to share with PhotoHunt, which is in paginated form.

Note: The get_and_store_friends() method does not request all pages of a user's friends, but rather just the first page. Your app might choose to page through the people.list calls by setting the pageToken parameter to the value of the current call's nextPageToken property. In this way, your app can get the user's entire friends list. Because this task could take some time, you would ideally perform the requests as an offline or asynchronous job.
Interactive posts

Interactive posts allow Alice to promote her photos that she uploads to PhotoHunt, and to invite her friends to PhotoHunt.

Important: Interactive posts will not work when PhotoHunt is hosted at http://localhost:8080 because the Google crawler can only access public URLs to get microdata about the content of the post. In the case of PhotoHunt for Python, you can deploy your app to appspot.com as a public Google App Engine app.
First, a button is added to static/index.html to allow Alice to invite her friends.

<button id="invite" class="button icon add primary" ng-show="themes">
  Invite your friends
</button>
This invite button is rendered as an interactive post button in the $scope.start function in the static/index.html file. This function runs each time that the index page is loaded:

$scope.start = function() {
  $scope.renderSignIn();
  $scope.checkForHighlightedPhoto();
  PhotoHuntApi.getThemes().then(function(response) {
    $scope.themes = response.data;
    $scope.selectedTheme = $scope.themes[0];
    $scope.orderBy('recent');
    $scope.getUserPhotos();
    var options = {
      'clientid': Conf.clientId,
      'contenturl': Conf.rootUrl + '/invite.html',
      'contentdeeplinkid': '/',
      'prefilltext': 'Join the hunt, upload and vote for photos of ' +
          $scope.selectedTheme.displayName + '. #photohunt',
      'calltoactionlabel': 'Join',
      'calltoactionurl': Conf.rootUrl,
      'calltoactiondeeplinkid': '/',
      'requestvisibleactions': Conf.requestvisibleactions,
      'scope': Conf.scopes,
      'cookiepolicy': Conf.cookiepolicy
    };
    gapi.interactivepost.render('invite', options);
    $scope.getAllPhotos();
  });
}
You can see all of the options that are provided to the gapi.interactivepost.render() method and importantly the contentUrl property. This property is important because that is the URL where Google crawls to create a share snippet for the Google+ stream such as the title for the page and the thumbnail to use. In this case, Google is directed to crawl invite.html which renders a simple page containing the metadata for the invite interactive post. If a user browses to that URL directly, the app redirects them to the index page by setting the window.location.href property using JavaScript. When Google crawls the invite.html page for metadata to display in the interactive post, the crawler will not execute JavaScript and so will not be redirected.

<!DOCTYPE html>
<html>
<head>
  <script type="text/javascript">
    window.location.href = '{{redirectUrl}}';
  </script>
  <title>{{name}}</title>
</head>
<body itemscope itemtype="http://schema.org/Thing">
  <h1 itemprop="name">{{name}}</h1>
  <img itemprop="image" src="{{imageUrl}}" />
  <p itemprop="description">{{description}}</p>
</body>
</html>
This file is rendered as a Jinja2 template by SchemaHandler.get():

def get(self):
  """Returns the template at templates/${request.path}.

     Issues the following errors along with corresponding HTTP response codes:
     404: 'Not Found'. No template was found for the specified path.
  """
  try:
    photo_id = self.request.get('photoId')
    self.response.headers['Content-Type'] = 'text/html'
    template = jinja_environment.get_template('templates' + self.request.path)
    if photo_id:
      photo = model.Photo.get_by_id(long(photo_id))
      self.response.out.write(template.render({
        'photoId': photo_id,
        'redirectUrl': 'index.html?photoId={}'.format(photo_id),
        'name': 'Photo by {} for {} | PhotoHunt'.format(
            photo.owner_display_name,
            photo.theme_display_name),
        'imageUrl': photo.thumbnail_url,
        'description': '{} needs your vote to win this hunt.'.format(
            photo.owner_display_name)
      }))
    else:
      photo = model.Photo.all().get()
      if photo:
        self.response.out.write(template.render({
          'redirectUrl': 'index.html?photoId='.format(photo_id),
          'name': 'Photo by {} for {} | PhotoHunt'.format(
              photo.owner_display_name,
              photo.theme_display_name),
          'imageUrl': photo.thumbnail_url,
          'description': 'Join in the PhotoHunt game.'
        }))
      else:
        self.response.out.write(template.render({
          'redirectUrl': get_base_url(),
          'name': 'PhotoHunt',
          'imageUrl': '{}/images/interactivepost-icon.png'.format(
              get_base_url()),
          'description': 'Join in the PhotoHunt game.'
        }))
  except TypeError as te:
    self.send_error(404, "Resource not found")
Next, a button is added to each photo that PhotoHunt lists, allowing Alice to promote photos. This addition is done in an AngularJS partial, which is located in the static/partials/photo.html file.

<button class="button">Promote</button>
Similarly to the invite button, the promote button is made useful with JavaScript, but this time it is made so using an AngularJS directive while rendering the static/partials/photo.html partial for each photo that is listed on the page. The relevant JavaScript is in the static/js/directives.js file.

angular.module('photoHunt.directives', ['photoHunt.services'])
    .directive('photo', function(Conf, PhotoHuntApi) {
      return {
        restrict: 'E',
        replace: true,
        scope: {
          item: '=',
          deletePhoto: '&deletePhoto'
        },
        templateUrl: 'partials/photo.html',
        link: function (scope, element, attrs) {
          element.find('.voteButton')
              .click(function(evt) {
                if(scope.item.canVote && !scope.item.voted) {
                  var voteButton = angular.element(evt.target)
                  scope.$apply(function() {
                    voteButton.unbind('click');
                    scope.item.numVotes = scope.item.numVotes + 1;
                    scope.item.voted = true;
                    voteButton.focus();
                    scope.item.voteClass.push('disable');
                  });
                  PhotoHuntApi.votePhoto(scope.item.id)
                      .then(function(response) {});
                }
              });

          element.find('.remove')
              .click(function() {
                if (scope.item.canDelete) {
                  scope.deletePhoto({photoId: scope.item.id});
                }
              });

          var options = {
            'clientid': Conf.clientId,
            'contenturl': scope.item.photoContentUrl,
            'contentdeeplinkid': '/?id=' + scope.item.id,
            'prefilltext': 'What do you think?  Does this image embody \'' +
                scope.item.themeDisplayName + '\'? #photohunt',
            'calltoactionlabel': 'VOTE',
            'calltoactionurl': scope.item.voteCtaUrl,
            'calltoactiondeeplinkid': '/?id=' + scope.item.id + '&action=VOTE',
            'requestvisibleactions': Conf.requestvisibleactions,
            'scope': Conf.scopes,
            'cookiepolicy': Conf.cookiepolicy
          }
          gapi.interactivepost.render(
              element.find('.toolbar button').get(0), options);
        }
      }
    })
Notice that the options provided for the promote interactive post are similar to the options that are provided for the invite interactive post. As with the invite interactive post, this time we provide a contentUrl, but its value is http://localhost:8080/photo.html?photoId=0000. The SchemaHandler behind the scenes of this HTML page also generates schema.org microdata, but this time the microdata is for the particular photo that is being requested.

After these interactive posts are created and shared by Alice, Bob clicks the "Vote" button in her promotion post, while Charles clicks the "Join" button in his invitation post. Bob and Charles go to the respective call-to-action URLs that are provided in the interactive post options.

App activities

The last part of Alice's use of PhotoHunt that involves the Google+ Platform is when Alice takes an action such as voting or uploading a photo. PhotoHunt writes that app activity to Google, enabling Google to show the information to other users when it is most relevant.

Important: App activities will not work when PhotoHunt is hosted at http://localhost:8080 because the Google crawler can only access public URLs to get microdata about the content of the activity. In the case of PhotoHunt for Python, you can deploy your app to appspot.com as a public Google App Engine app.
In Alice's case, this is done when she uploads a new photo. The post() method of PhotosHandler, exposed as http://localhost:8080/api/photos, makes a call to add_photo_to_google_plus_activity(). This method creates an activity JSON object and sends it to the moments.insert method to write the activity to Alice's profile.

def add_photo_to_google_plus_activity(self, user, photo):
  """Creates an app activity in Google indicating that the given User has
  uploaded the given Photo.

  Args:
    user: Creator of Photo.
    photo: Photo itself.
  """
  activity = {"type":"http://schema.org/AddAction",
            "target": {
              "url": photo.photo_content_url
            }}
  logging.debug("activity: " + str(activity))
  http = httplib2.Http()
  plus = build('plus', 'v1', http=http)
  if user.google_credentials:
    http = user.google_credentials.authorize(http)
  return plus.moments().insert(userId='me', collection='vault',
                                  body=activity).execute()
Disconnecting from PhotoHunt

The developer policies require that apps that use the Google APIs provide the ability for users to disconnect and that apps must delete all of the data that they collect about a user from the Google+ API.

This operation is handled in DisconnectHandler, in the post() method. The method:

Deletes all of the friend relationships that Alice shared with PhotoHunt from her Google+ circles.
Deletes all of Alice's votes and photos (optional).
Deletes the User object that represents Alice. This is done to delete all of Alice's Google information, but otherwise deleting the entire User object is not necessary in your app.
Disconnects the app from Alice's account by revoking all of the authorization tokens that are issued to PhotoHunt for Alice. This step revokes tokens across the web, Android, and iOS clients.

def post(self):
  """Exposed as `POST /api/disconnect`.

  As required by the Google+ Platform Terms of Service, this end-point:

    1. Deletes all data retrieved from Google that is stored in our app.
    2. Revokes all of the user's tokens issued to this app.

  Takes no request payload, and disconnects the user currently identified
  by their session.

  Returns the following JSON response representing the User that was
  connected:

    'Successfully disconnected.'

  Issues the following errors along with corresponding HTTP response codes:
  401: 'Unauthorized request'.  No user was connected to disconnect.
  500: 'Failed to revoke token for given user: '
       + error from failed connection to revoke end-point.
  """
  try:
    user = self.get_user_from_session()
    credentials = user.google_credentials

    del(self.session[self.CURRENT_USER_SESSION_KEY])
    user_id = user.key().id()
    db.delete(model.Vote.all().filter("owner_user_id =", user_id).run())
    db.delete(model.Photo.all().filter("owner_user_id =", user_id).run())
    db.delete(model.DirectedUserToUserEdge.all().filter(
        "owner_user_id =", user_id).run())
    db.delete(user)

    DisconnectHandler.revoke_token(credentials)
    self.send_success('Successfully disconnected.')
    return
  except UserNotAuthorizedException as e:
    self.send_error(401, e.msg)
    return
  except RevokeException as e:
    self.send_error(500, e.msg)
    return

def revoke_token(credentials):
  """Revoke the given access token, and consequently any other access tokens
  and refresh tokens issued for this user to this app.

  Essentially this operation disconnects a user from the app, but keeps
  their app activities alive in Google.  The same user can later come back
  to the app, sign-in, re-consent, and resume using the app.
  throws RevokeException error occured while making request.
  """
  url = TOKEN_REVOKE_ENDPOINT % credentials.access_token
  http = httplib2.Http()
  credentials.authorize(http)
  result = http.request(url, 'GET')[0]

  if result['status'] != '200':
    raise RevokeException
Reusing the pieces for other users

Bob's interaction with PhotoHunt

Bob's flow touches interactive posts, the Google+ Sign-In flow (note that this is different from the button itself), app personalization, and app activities. His flow touches these pieces in the given order.

As a reminder, Bob is not currently a PhotoHunt user and his interaction with the app is:

Bob views a notification of Alice's interactive share post.
He clicks the Vote button in the interactive post.
Bob signs in with Google+ to create an account on PhotoHunt.
When Bob sees the notification of the share from Alice, and clicks the Vote button, he is actually clicking a link to the call-to-action URL if Bob is originating from web, or he is clicking a deep link to the call-to-action deep link ID if he is originating from Android or iOS.

Next, Bob is taken to PhotoHunt and clicks the Google+ Sign-In button. After signing in, Bob's vote can be recorded. Although PhotoHunt doesn't do this, your app could require the user to sign-in after landing on the page from an interactive post.

Charles' invite

Charles' flow touches the same pieces of the Google+ platform as Bob's flow, but for a different reason, which is to join PhotoHunt instead of to vote on a photo. Otherwise, the case is already supported by the same code we wrote for Alice.

David's photos

When David signed in to PhotoHunt for the first time he chose to give PhotoHunt access to his circles, which include Alice. What is important to understand about the relationship between Alice and David is that David's photos are shown to Alice in her "Photos by Friends" section if and only if the following conditions are met:

David has Alice in his Google+ circles
David has chosen to tell PhotoHunt that he has Alice in his Google+ circles
Alice has David in her Google+ circles
Alice has chosen to tell PhotoHunt that she has David in her Google+ circles
Connections between two users in Google+ are independent of each other. For example, Alice might have David in her circles but David might not have Alice in his circles. Your app should respect the user's preferences and not leak data to people who your user does not want to share with.

Next steps

After you are done working through PhotoHunt, and you thoroughly understand how each feature is implemented, you should spend some time applying each feature to your own app. PhotoHunt is intentionally written as a complete end-to-end application so that you can adapt the concepts easily to your own app.

As you integrate your app with the Google+ Platform, you will likely need to make use of the API reference or the Android or iOS mobile SDKs.
