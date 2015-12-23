# Updating user profiles
The last feature we will implement in this tutorial is the ability for a user to update their profile. The updates we offer will be minimal, including updating the user's first name, last name, email, and tagline, but you will get the gist of it and can add more options at will.

{video: update-user-profile}


## ProfileSettingsController
To get started, open `static/javascripts/profiles/controllers/profile-settings.controller.js` and add the following contents:

    /**
    * ProfileSettingsController
    * @namespace thinkster.profiles.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.profiles.controllers')
        .controller('ProfileSettingsController', ProfileSettingsController);

      ProfileSettingsController.$inject = [
        '$location', '$routeParams', 'Authentication', 'Profile', 'Snackbar'
      ];

      /**
      * @namespace ProfileSettingsController
      */
      function ProfileSettingsController($location, $routeParams, Authentication, Profile, Snackbar) {
        var vm = this;

        vm.destroy = destroy;
        vm.update = update;

        activate();


        /**
        * @name activate
        * @desc Actions to be performed when this controller is instantiated.
        * @memberOf thinkster.profiles.controllers.ProfileSettingsController
        */
        function activate() {
          var authenticatedAccount = Authentication.getAuthenticatedAccount();
          var username = $routeParams.username.substr(1);

          // Redirect if not logged in
          if (!authenticatedAccount) {
            $location.url('/');
            Snackbar.error('You are not authorized to view this page.');
          } else {
            // Redirect if logged in, but not the owner of this profile.
            if (authenticatedAccount.username !== username) {
              $location.url('/');
              Snackbar.error('You are not authorized to view this page.');
            }
          }

          Profile.get(username).then(profileSuccessFn, profileErrorFn);

          /**
          * @name profileSuccessFn
          * @desc Update `profile` for view
          */
          function profileSuccessFn(data, status, headers, config) {
            vm.profile = data.data;
          }

          /**
          * @name profileErrorFn
          * @desc Redirect to index
          */
          function profileErrorFn(data, status, headers, config) {
            $location.url('/');
            Snackbar.error('That user does not exist.');
          }
        }


        /**
        * @name destroy
        * @desc Destroy this user's profile
        * @memberOf thinkster.profiles.controllers.ProfileSettingsController
        */
        function destroy() {
          Profile.destroy(vm.profile).then(profileSuccessFn, profileErrorFn);

          /**
          * @name profileSuccessFn
          * @desc Redirect to index and display success snackbar
          */
          function profileSuccessFn(data, status, headers, config) {
            Authentication.unauthenticate();
            window.location = '/';

            Snackbar.show('Your account has been deleted.');
          }


          /**
          * @name profileErrorFn
          * @desc Display error snackbar
          */
          function profileErrorFn(data, status, headers, config) {
            Snackbar.error(data.error);
          }
        }


        /**
        * @name update
        * @desc Update this user's profile
        * @memberOf thinkster.profiles.controllers.ProfileSettingsController
        */
        function update() {
          Profile.update(vm.profile).then(profileSuccessFn, profileErrorFn);

          /**
          * @name profileSuccessFn
          * @desc Show success snackbar
          */
          function profileSuccessFn(data, status, headers, config) {
            Snackbar.show('Your profile has been updated.');
          }


          /**
          * @name profileErrorFn
          * @desc Show error snackbar
          */
          function profileErrorFn(data, status, headers, config) {
            Snackbar.error(data.error);
          }
        }
      }
    })();

{x: profile_settings_controller}
Create the `ProfileSettingsController` controller

Be sure to include this file in `javascripts.html`:

    <script type="text/javascript" src="{% static 'javascripts/profiles/controllers/profile-settings.controller.js' %}"></script>

{x: profile_settings_controller_include_js}
Include `profile-settings.controller.js` in `javascripts.html`

Here we have created two methods that will be available to the view: `update` and `destroy`. As their names suggest, `update` will allow the user to update their profile and `destroy` will destroy the user's account.

Most of this controller should look familiar, but let's go over the methods we've created for clarity.

    /**
     * @name activate
     * @desc Actions to be performed when this controller is instantiated.
     * @memberOf thinkster.profiles.controllers.ProfileSettingsController
     */
    function activate() {
      var authenticatedAccount = Authentication.getAuthenticatedAccount();
      var username = $routeParams.username.substr(1);

      // Redirect if not logged in
      if (!authenticatedAccount) {
        $location.url('/');
        Snackbar.error('You are not authorized to view this page.');
      } else {
        // Redirect if logged in, but not the owner of this profile.
        if (authenticatedAccount.username !== username) {
          $location.url('/');
          Snackbar.error('You are not authorized to view this page.');
        }
      }

      Profile.get(username).then(profileSuccessFn, profileErrorFn);

      /**
       * @name profileSuccessFn
       * @desc Update `profile` for view
       */
      function profileSuccessFn(data, status, headers, config) {
        vm.profile = data.data;
      }

      /**
       * @name profileErrorFn
       * @desc Redirec to index
       */
      function profileErrorFn(data, status, headers, config) {
        $location.url('/');
        Snackbar.error('That user does not exist.');
      }
    }


In `activate`, we follow a familiar pattern. Because this page allows for dangerous operations to be performed, we must make sure the current user is authorized to see this page. We do this by first checking if the user is authenticated and then checking if the authenticated user owns the profile. If either case is false, then we redirect to the index page with a snackbar error stating that the user is not authorized to view this page.

If the authorization process succeeds, we simply grab the user's profile from the server and allow the user to do as they wish.

    /**
     * @name destroy
     * @desc Destroy this user's profile
     * @memberOf thinkster.profiles.controllers.ProfileSettingsController
     */
    function destroy() {
      Profile.destroy(vm.profile).then(profileSuccessFn, profileErrorFn);

      /**
       * @name profileSuccessFn
       * @desc Redirect to index and display success snackbar
       */
      function profileSuccessFn(data, status, headers, config) {
        Authentication.unauthenticate();
        window.location = '/';

        Snackbar.show('Your account has been deleted.');
      }


      /**
       * @name profileErrorFn
       * @desc Display error snackbar
       */
      function profileErrorFn(data, status, headers, config) {
        Snackbar.error(data.error);
      }
    }

When a user wishes to destroy their profile, we must unauthenticate them and redirect to the index page, performing a page refresh in the process. This will make the navigation bar re-render with the logged out view.

If for some reason destroying the user's profile returns an error status code, we simply display an error snackbar with the error message returned by the server. We do not perform any other actions because we see no reason why this call should fail unless the user is not authorized to delete this profile, but we have already accounted for this scenario in the `activate` method.

    /**
     * @name update
     * @desc Update this user's profile
     * @memberOf thinkster.profiles.controllers.ProfileSettingsController
     */
    function update() {
      Profile.update(vm.profile).then(profileSuccessFn, profileErrorFn);

      /**
       * @name profileSuccessFn
       * @desc Show success snackbar
       */
      function profileSuccessFn(data, status, headers, config) {
        Snackbar.show('Your profile has been updated.');
      }


      /**
       * @name profileErrorFn
       * @desc Show error snackbar
       */
      function profileErrorFn(data, status, headers, config) {
        Snackbar.error(data.error);
      }
    }

`update()` is very simple. Whether the call succeeds or fails, we show a snackbar with the appropriate message.

## A template for the settings page
As usual, now that we have the controller we need to make a corresponding template.

Create `static/templates/profiles/settings.html` with the following content:

    <div class="col-md-4 col-md-offset-4">
      <div class="well" ng-show="vm.profile">
        <form role="form" class="settings" ng-submit="vm.update()">
          <div class="form-group">
            <label for="settings__email">Email</label>
            <input type="text" class="form-control" id="settings__email" ng-model="vm.profile.email" placeholder="ex. john@example.com" />
          </div>

          <div class="form-group">
            <label for="settings__password">New Password</label>
            <input type="password" class="form-control" id="settings__password" ng-model="vm.profile.password" placeholder="ex. notgoogleplus" />
          </div>

          <div class="form-group">
            <label for="settings__confirm-password">Confirm Password</label>
            <input type="password" class="form-control" id="settings__confirm-password" ng-model="vm.profile.confirm_password" placeholder="ex. notgoogleplus" />
          </div>

          <div class="form-group">
            <label for="settings__username">Username</label>
            <input type="text" class="form-control" id="settings__username" ng-model="vm.profile.username" placeholder="ex. notgoogleplus" />
          </div>

          <div class="form-group">
            <label for="settings__tagline">Tagline</label>
            <textarea class="form-control" id="settings__tagline" ng-model="vm.profile.tagline" placeholder="ex. This is Not Google Plus." />
          </div>

          <div class="form-group">
            <button type="submit" class="btn btn-primary">Submit</button>
            <button type="button" class="btn btn-danger pull-right" ng-click="vm.destroy()">Delete Account</button>
          </div>
        </form>
      </div>
    </div>

{x: profile_settings_html}
Create a template for `ProfileSettingsController`

This template is similar to the forms we created for registering and logging in. There is nothing here worth discussing. 

## Profile settings route
Open up `static/javascripts/thinkster.routes.js` and add the following route:

    // ...
    .when('/+:username/settings', {
      controller: 'ProfileSettingsController',
      controllerAs: 'vm',
      templateUrl: '/static/templates/profiles/settings.html'
    })
    // ...

## Checkpoint
And that's our last feature! You should now be able to load up the settings page at `http://localhost:8000/+:username/settings` and update your settings as you wish.

Try updating your tagline. If it works, you will now see your tagline displayed on your profile page.

{x: checkpoint_update_user_profile}
Update your tagline and view the new tagline on your profile page
