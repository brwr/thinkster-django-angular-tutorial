# Logging users out
Given that users can register and login, we can assume they will want a way to log out. People get mad when they can't log out.

{video: log-user-out}

## Making a logout API view
Let's implement the last authentication-related API view.

Open up `authentication/views.py` and add the following imports and class:

    from django.contrib.auth import logout

    from rest_framework import permissions

    class LogoutView(views.APIView):
        permission_classes = (permissions.IsAuthenticated,)

        def post(self, request, format=None):
            logout(request)

            return Response({}, status=status.HTTP_204_NO_CONTENT)

{x: django_logout_view}
Make a view called `LogoutView` in `authentication/views.py`

There are only a few new things to talk about this time.

    permission_classes = (permissions.IsAuthenticated,)

Only authenticated users should be able to hit this endpoint. Django REST Framework's `permissions.IsAuthenticated` handles this for us. If you user is not authenticated, they will get a `403` error.

    logout(request)

If the user is authenticated, all we need to do is call Django's `logout()` method.

    return Response({}, status=status.HTTP_204_NO_CONTENT)

There isn't anything reasonable to return when logging out, so we just return an empty response with a `200` status code.

Moving on to the URLs.

Open up `thinkster_django_angular_boilerplate/urls.py` again and add the following import and URL:

    from authentication.views import LogoutView

    urlpatterns = patterns(
        # ...
        url(r'^api/v1/auth/logout/$', LogoutView.as_view(), name='logout'),
        #...
    )

{x: django_url_logout}
Create an API endpoint for `LogoutView`

## Logout: AngularJS Service
The final method you need to add to your `Authentication` service is the `logout()` method.

Add the following method to the `Authentication` service in `authentication.service.js`:

    /**
     * @name logout
     * @desc Try to log the user out
     * @returns {Promise}
     * @memberOf thinkster.authentication.services.Authentication
     */
    function logout() {
      return $http.post('/api/v1/auth/logout/')
        .then(logoutSuccessFn, logoutErrorFn);

      /**
       * @name logoutSuccessFn
       * @desc Unauthenticate and redirect to index with page reload
       */
      function logoutSuccessFn(data, status, headers, config) {
        Authentication.unauthenticate();

        window.location = '/';
      }

      /**
       * @name logoutErrorFn
       * @desc Log "Epic failure!" to the console
       */
      function logoutErrorFn(data, status, headers, config) {
        console.error('Epic failure!');
      }
    }

As always, remember to expose `logout` as part of the `Authentication` service:

    var Authentication = {
      getAuthenticatedUser: getAuthenticatedUser,
      isAuthenticated: isAuthenticated,
      login: login,
      logout: logout,
      register: register,
      setAuthenticatedUser: setAuthenticatedUser,
      unauthenticate: unauthenticate
    };

{x: angularjs_authentication_service_logout}
Add a `logout()` method to your `Authentication` service

## Controlling the navigation bar with NavbarController
There will not actually be a `LogoutController` or `logout.html`. Instead, the navigation bar already contains a logout link for authenticated users. We will create a `NavbarController` for handling the logout buttons `onclick` functionality and we will update the link itself with an `ng-click` attribute.

Create a file in `static/javascripts/layout/controllers/` called `navbar.controller.js` and add the following to it:

    /**
    * NavbarController
    * @namespace thinkster.layout.controllers
    */
    (function () {
      'use strict';

      angular
        .module('thinkster.layout.controllers')
        .controller('NavbarController', NavbarController);

      NavbarController.$inject = ['$scope', 'Authentication'];

      /**
      * @namespace NavbarController
      */
      function NavbarController($scope, Authentication) {
        var vm = this;

        vm.logout = logout;

        /**
        * @name logout
        * @desc Log the user out
        * @memberOf thinkster.layout.controllers.NavbarController
        */
        function logout() {
          Authentication.logout();
        }
      }
    })();

{x: angularjs_navbar_controller}
Create a `NavbarController` in `static/javascripts/layout/controllers/navbar.controller.js`

Open `templates/navbar.html` and add an `ng-controller` directive with the value `NavbarController as vm` to the `<nav />` tag like so:

    <nav class="navbar navbar-default" role="navigation" ng-controller="NavbarController as vm">

While you have `templates/navbar.html` open, go ahead and find the logout link and add `ng-click="vm.logout()"` to it like so:

    <li><a href="javascript:void(0)" ng-click="vm.logout()">Logout</a></li>

{x: angularjs_navbar_template_update}
Update `navbar.html` to include the `ng-controller` and `ng-click` directives where appropriate

## Layout modules
We need to add a few new modules this time around.

Create a file in `static/javascripts/layout/` called `layout.module.js` and give it the following contents:

    (function () {
      'use strict';

      angular
        .module('thinkster.layout', [
          'thinkster.layout.controllers'
        ]);

      angular
        .module('thinkster.layout.controllers', []);
    })();


And don't forget to update `static/javascripts/thinkster.js` also:

    angular
      .module('thinkster', [
        'thinkster.config',
        'thinkster.routes',
        'thinkster.authentication',
        'thinkster.layout'
      ]);

{x: angularjs_static_module}
Define new `thinkster.layout` and `thinkster.layout.controllers` modules

## Including new .js files
This time around there are a couple new JavaScript files to include. Open up `javascripts.html` and add the following:

    <script type="text/javascript" src="{% static 'javascripts/layout/layout.module.js' %}"></script>
    <script type="text/javascript" src="{% static 'javascripts/layout/controllers/navbar.controller.js' %}"></script>

{x: include_javascript_static}
Include new JavaScript files in `javascripts.html`

## Checkpoint
If you visit `http://localhost:8000/` in your browser, you should still be logged in. If not, you will need to log in again.

You can confirm the logout functionality is working by clicking the logout button in the navigation bar. This should refresh the page and update the navigation bar to it's logged out view.

{x: checkpoint_logout}
Log out of your account by using the logout button in the navigation bar
