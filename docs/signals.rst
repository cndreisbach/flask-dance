.. module:: flask_dance.consumer

Signals
=======

.. versionadded:: 0.2

Flask-Dance supports signals, :ref:`just as Flask does <flask:signals>`.
Signals are perfect for custom processing code that you want to run at a certain
point in the OAuth dance. For example, after the dance is complete, you might
need to update the user's profile, kick off a long-running task, or simply
:ref:`flash a message <flask:message-flashing-pattern>` to let the user know
that the login was successful. It's easy, just import the appropriate signal of
the ones listed below, and connect your custom processing code to the signal.

Core Signals
------------
The following signals exist in Flask-Dance:

.. data:: oauth_authorized

    This signal is sent when a user completes the OAuth dance by receiving a
    response from the OAuth provider's authorize URL. This signal is sent
    regardless of whether the authorization was successful: the user may
    have declined the authorization request, for example. The signal is invoked
    with the blueprint instance as the first argument (the *sender*), and with
    a dict of the OAuth provider's response (the *token*).

    Example subscriber::

        from flask import flash
        from flask_dance.consumer import oauth_authorized

        @oauth_authorized.connect
        def logged_in(blueprint, token):
            if "error" in token:
                flash("You denied the request to sign in. Please try again.")
                del blueprint.token
            else:
                flash("Signed in successfully with {name}!".format(
                    name=blueprint.name.capitalize()
                ))

    If you're using OAuth 2, the user may grant you different scopes from the
    ones you requested: check the ``scope`` key in the *token* dict to
    determine what scopes were actually granted. If you don't want the *token*
    to be :doc:`stored <token-storage>`, simply return ``False`` from one of your
    signal receiver functions -- this can be useful if the user has declined
    to authorize your OAuth request, has granted insufficient scopes, or in some
    other way has given you a token that you don't want.


.. _flash a message: http://flask.pocoo.org/docs/latest/patterns/flashing/
.. _blinker: http://pythonhosted.org/blinker/
