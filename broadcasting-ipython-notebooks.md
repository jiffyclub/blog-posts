A useful feature of the [IPython Notebook][] is that you can set the server to
broadcast so that others on your local network can see the server and your
notebooks. This is especially nice as a teacher so that students can load
your notebooks as you work, copy text out of them, and see them in their
entirety instead of just what you have on screen. Here's the outline of what
to do, with detailed instructions below:

1. Create an IPython profile with a password for the Notebook server.
2. Figure out your IP address on the local network.
3. Launch IPython in broadcast + read-only mode using your new profile.
4. Have your students navigate to your Notebook server.

# Make a New Profile

IPython has quite a number of [configuration options][config] that you might
like to be set one way or another depending what you're doing. To help make
that configuration management easier IPython has [profiles][]. We're going to
make a new profile in which we'll set a password that must be entered to edit
or make new notebooks. Here's the command:

    ipython profile create teaching

I'll use a profile named "teaching" throughout these instructions, you can name
it whatever you like.

# Setting a Notebook Password

One of IPython's configuration options is the ability to set a password on the
Notebook server. This usually makes it so that only people with the password
can access the server, but if you start the server with the `--read-only` flag
anyone can access and see the notebooks, but only you with the password can
can edit notebooks or make new ones.

Follow the IPython instructions on [making a Notebook password][security] to
generate your password hash string. Once you've got your password hash string
(which should look something like
`sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed`), you're going to
put that into the Notebook configuration file for you new teaching profile. To
find out where that is you can run this command:

    ipython locate profile teaching

It will probably tell you something like `~/.ipython/profile_teaching`, which
is the directory where the configuration files for that profile are stored.
Open the file called `ipython_notebook_config.py` within that directory and
find the line that has the string `c.NotebookApp.password`. Uncomment that line
and paste your password hash string into the empty quotes there. When you're
done it should like this:

    c.NotebookApp.password = u'sha1:67c9e60bb8b6:9ffede0825894254b2e042ea597d771089e11aed'

And when you start the Notebook server with the teaching profile it will be
password protected. You can change your password at any time be generating a
new password hash string and pasting into `ipython_notebook_config.py`.

# Find Out Your IP Address

Your students need to know where to point their browsers so they can see your
notebooks and for that you need to know your IP address on the local network.
On Macs you can get this from the Network pane of System Preferences, but I
like to use [this Python script from Software Carpentry][get-my-ip], which
should work on any system. On my home WiFi my IP is usually something like
`10.0.1.9` but it will vary depending on where you are. Take note of your IP
address, in a minute we'll combine it with the port number used by the IPython
Notebook server so your students can type something into their URL bars.

# Start The Notebook Server and Note the Port Number

It's time to start your Notebook server with some added options. You're going
to tell it:

* Use my teaching profile: `--profile=teaching`
* Broadcast yourself: `--ip='*'`
* Allow others to see my notebooks without a password: `--read-only`

Here's what the whole command looks like:

    ipython notebook --profile=teaching --read-only --ip='*'

When IPython starts up the Notebook server it will print a line containing
the URL of the server, including the port number. It will look like this:

    [NotebookApp] The IPython Notebook is running at: http://[all ip addresses on your system]:8888/

The last four numbers after the colon (in this case `8888`) are the port number
of the server. Take note of that number.

# Tell Your Students Where to Go

Finally you can tell your students what to type into their browsers. Combine
your IP address with the Notebook server port number (separated by a colon)
and that will be the URL your students go to. For me that would be:

    http://10.0.1.9:8888/

When your students land there they should see the usual IPython Notebook
dashboard and they should be able to look at your existing notebooks, but they
shouldn't be able to create new notebooks, save changes to your notebooks, or
execute code.

# Some Notes

* The students' views don't update in real time. I find that in order for them
    to see my changes I must save my notebook and then the students have to
    manually reload. They won't see unsaved changes and they won't see any changes
    at all unless they reload.
* The [IPython security docs][security] recommend making an SSL certificate so
    you can connect to your server over HTTPS, which will encrypt your password.
    I generally don't bother, but be advised that anyone who can execute code
    on your computer can do things like delete your entire hard drive.
    Either way, be sure not to use a password you use anywhere else.

[IPython Notebook]: http://ipython.org/notebook.html
[config]: http://ipython.org/ipython-doc/dev/config/overview.html
[profiles]: http://ipython.org/ipython-doc/dev/config/overview.html#profiles
[security]: http://ipython.org/ipython-doc/dev/interactive/htmlnotebook.html#security
[get-my-ip]: https://github.com/swcarpentry/boot-camps/blob/master/setup/get-my-ip.py
