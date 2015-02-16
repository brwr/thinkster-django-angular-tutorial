# Displaying user profiles

{video: display-user-profile}

We already have the Django views and routes necessary to display a profile for each user. From here we can jump into making an AngularJS service and then move on to the template and controllers.

{info}
In this section and the next, we will refer to accounts as profiles. For the purposes of our client, that is effectively what the `Account` model translates into: a user's profile.


## Making the profile modules
We will be creating a service and a couple of controllers relating to user profiles, so let's go ahead and define the modules we will need.

Create `static/javascripts/profiles/profiles.module.js` with the following content:

    (function () {
      'use strict';

      angular
        .module('thinkster.profiles', [
          'thinkster.profiles.controllers',
          'thinkster.profiles.services'
        ]);

      angular
        .module('thinkster.profiles.controllers', []);

      angular
        .module('thinkster.profiles.services', []);
    })();

{x: profile_modules}
Define the modules needed for profiles in `profiles.module.js`

As always, don't forget to register `thinkster.profiles` as a dependency of `thinkster` in `thinkster.js`:

    angular
      .module('thinkster', [
        'thinkster.config',
        'thinkster.routes',
        'thinkster.authentication',
        'thinkster.layout',
        'thinkster.posts',
        'thinkster.profiles'
      ]);

{x: thinkster_profile_module}
Register `thinkster.profiles` as a dependency of the `thinkster` module

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/profiles/profiles.module.js' %}"></script>

## Making a Profile factory
With the module definitions in place, we are ready to create the `Profile` service that will communicate with our API.

Create `static/javascripts/profiles/services/profile.service.js` with the following contents:

    /**
    * Profile
    * @namespace thinkster.profiles.services
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.profiles.services')
        .factory('Profile', Profile);

      Profile.$inject = ['$http'];

      /**
      * @namespace Profile
      */
      function Profile($http) {
        /**
        * @name Profile
        * @desc The factory to be returned
        * @memberOf thinkster.profiles.services.Profile
        */
        var Profile = {
          destroy: destroy,
          get: get,
          update: update
        };

        return Profile;

        /////////////////////

        /**
        * @name destroy
        * @desc Destroys the given profile
        * @param {Object} profile The profile to be destroyed
        * @returns {Promise}
        * @memberOf thinkster.profiles.services.Profile
        */
        function destroy(profile) {
          return $http.delete('/api/v1/accounts/' + profile.username + '/');
        }


        /**
        * @name get
        * @desc Gets the profile for user with username `username`
        * @param {string} username The username of the user to fetch
        * @returns {Promise}
        * @memberOf thinkster.profiles.services.Profile
        */
        function get(username) {
          return $http.get('/api/v1/accounts/' + username + '/');
        }


        /**
        * @name update
        * @desc Update the given profile
        * @param {Object} profile The profile to be updated
        * @returns {Promise}
        * @memberOf thinkster.profiles.services.Profile
        */
        function update(profile) {
          return $http.put('/api/v1/accounts/' + profile.username + '/', profile);
        }
      }
    })();

{x: profiles_factory}
Create a new factory called `Profiles` in `static/javascripts/profiles/services/profiles.service.js`

We aren't doing anything special here. Each of these API calls is a basic CRUD operation, so we get away with not having much code.

Add this file to `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/profiles/services/profile.service.js' %}"></script>

{x: profiles_service_include_js}
Include `profiles.service.js` in `javascripts.html`

## Making an interface for user profiles
Create `static/templates/profiles/profile.html` with the following content:

    <div class="profile" ng-show="vm.profile">
      <div class="jumbotron profile__header">
        <h1 class="profile__username">+{{ vm.profile.username }}</h1>
        <p class="profile__tagline">{{ vm.profile.tagline }}</p>
      </div>

      <posts posts="vm.posts"></posts>
    </div>

{x: profile_template}
Make a template for displaying profiles in `static/templates/profiles/profile.html`

This will render a header with the username and tagline of the profile owner, followed by a list of their posts. The posts are rendered using the directive we created earlier for the index page.

## Controlling the profile interface with ProfileController
The next step is to create the controller that will use the service we just created, along with the `Post` service, to retrieve the data we want to display.

Create `static/javascripts/profiles/controllers/profile.controller.js` with the following content:

    /**
    * ProfileController
    * @namespace thinkster.profiles.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.profiles.controllers')
        .controller('ProfileController', ProfileController);

      ProfileController.$inject = ['$location', '$routeParams', 'Posts', 'Profile', 'Snackbar'];

      /**
      * @namespace ProfileController
      */
      function ProfileController($location, $routeParams, Posts, Profile, Snackbar) {
        var vm = this;

        vm.profile = undefined;
        vm.posts = [];

        activate();

        /**
        * @name activate
        * @desc Actions to be performed when this controller is instantiated
        * @memberOf thinkster.profiles.controllers.ProfileController
        */
        function activate() {
          var username = $routeParams.username.substr(1);

          Profile.get(username).then(profileSuccessFn, profileErrorFn);
          Posts.get(username).then(postsSuccessFn, postsErrorFn);

          /**
          * @name profileSuccessProfile
          * @desc Update `profile` on viewmodel
          */
          function profileSuccessFn(data, status, headers, config) {
            vm.profile = data.data;
          }


          /**
          * @name profileErrorFn
          * @desc Redirect to index and show error Snackbar
          */
          function profileErrorFn(data, status, headers, config) {
            $location.url('/');
            Snackbar.error('That user does not exist.');
          }


          /**
            * @name postsSucessFn
            * @desc Update `posts` on viewmodel
            */
          function postsSuccessFn(data, status, headers, config) {
            vm.posts = data.data;
          }


          /**
            * @name postsErrorFn
            * @desc Show error snackbar
            */
          function postsErrorFn(data, status, headers, config) {
            Snackbar.error(data.data.error);
          }
        }
      }
    })();

{x: profile_controller}
Create a new controller called `ProfileController` in `static/javascripts/profiles/controllers/profile.controller.js`

Include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/profiles/controllers/profile.controller.js' %}"></script>

{x: profile_controller_include_js}
Include `profile.controller.js` in `javascripts.html`

## Making a route for viewing user profiles
Open `static/javascripts/thinkster.routes.js` and add the following route:

    .when('/+:username', {
      controller: 'ProfileController',
      controllerAs: 'vm',
      templateUrl: '/static/templates/profiles/profile.html'
    })

{x: profile_route}
Make a route for viewing user profiles

## Checkpoint
To view your profile, direct your browser to `http://localhost:8000/+<username>`. If the page renders, everything is good!

{x: checkpoint_display_user_profile}
Visit your profile page at `http://localhost:8000/+<username>`
