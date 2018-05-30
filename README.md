﻿# JADA documentation #
	Version: V1.05-30-18.04.00P
	Last Update: 05/30/2018
* Latest status: Uploaaded files to Git Hub

## Introduction ##
	Jada is the Platform that Eve Runs on.

### Goals ###

* simplicity
* little boilerplate
* minimal magic
* power
* multithreading
* automatic reloading
* extensibility

### Features ###

* multithreaded dispatch and the ability to connect to multiple networks at
  a time
* easy plugin development with automatic reloading and a simple hooking API

### Requirements ###

Eve runs on Python 2.6 and 2.7. Many of the plugins require
[lxml](http://lxml.de/).

## Table of contents ##

* [[Installation]]

    File can be started via the terminal window. place Eve.py in the Bin directory.

* [[Configuration]]

   #!/usr/bin/python
# Eve.py - A simple Python IRC bot
# Authors:  Jraglin
#Version: V1.02-27-17.12.52A  
# Last stable: Eve1-v1.02.26.17.23.27P.py
#---------------------------------------------------------------------
# Suggestions.

# These are just suggestions:
#
# 1. Limit lines to 70 characters (code or commands)
# 2. Indent using 4 spaces as opposed to tabs
# 3. Use some inline comments
# 4. Start to handle error cases
# 5. Change hard-coded constants to program parameters

#---------------------------------------------------------------------
# Overview.

# Eve is an IRC bot presently written in pure Python.  She is
# compatible with Python 2 but has not been tested with Python 3.

#---------------------------------------------------------------------
# Import required libraries.

import re, socket, os

#---------------------------------------------------------------------
# Some constants.

zero      = 0                   # Zero
one       = 1                   # One
false     = 0                   # Boolean False
true      = 1                   # Boolean True

#---------------------------------------------------------------------
# Program parameters.

# botnick  = Default nick
# bufsize  = Input buffer size
# channel  = Default IRC channel
# port     = Default IRC port number
# server   = Default IRC server hostname
# master   = Owner of the bot
# uname    = Bot username (NOT NICK!)
# realname = Bot's real name

botnick    = "Jynx2"
bufsize    = 2048
channel    = "#Jraglin"
port       = 8001
server     = "irc.opensuse.org"
master     = "jraglin"
uname      = "jynx2"
realname   = "Jynx2"

#---------------------------------------------------------------------
# Set up a brief-replies dictionary.

# This is used to implement brief replies to brief commands or remarks
# such as:
#
#     Jynx2, go away!
# or  Hello, Jynx2

Replies = dict()
Replies ['die'      ] = "No, you"
Replies ['goodbye'  ] = "I'll miss you"
Replies ['sayonara' ] = "I'll miss you"
Replies ['scram'    ] = "No, you"
Replies ['shout'    ] = "NO I WON'T"
Replies ['dance'    ] = botnick + " dances"
Replies ['sing'     ] = "Tra la la"
Replies ['hello'    ] = "Hi"
Replies ['howdy'    ] = "Hi"
Replies ['time'     ] = "It is TIME for a RHYME"
Replies ['master'   ] = master + " is my master"
Replies ['bacon'    ] = "Give me some, please!"
Replies [botnick    ] = "What do you want?"
# You can add more replies, like so:
# Replies['action] = "Reply"
#---------------------------------------------------------------------
# Subroutine.

# Name:       ping
# Arguments:  None
# Purpose:    Responds to server Pings

def ping():
    global ircsock
    ircsock.send ("PONG :pingis\n")

#---------------------------------------------------------------------
# Subroutine.

# Name:       sendmsg
# Arguments:  chan = channel
#             msg  = message
# Purpose:    Responds to server Pings

# Sends a specified message to a specified channel.

def sendmsg (chan, msg):
    global ircsock
    ircsock.send ("PRIVMSG "+ chan +" :"+ msg + "\n")

#---------------------------------------------------------------------
# Subroutine.

# Name:       JoinChan
# Arguments:  chan = channel
# Purpose:    Joins a channel

def JoinChan (chan):
    global ircsock
    ircsock.send ("JOIN "+ chan +"\n")

#---------------------------------------------------------------------
# Subroutine.

# Name:       ProcHello
# Arguments:  None
# Purpose:    Processes a Hello request

def ProcHello():
    global ircsock
    ircsock.send ("PRIVMSG "+ channel +" :Hello!\n")

#---------------------------------------------------------------------
# Main routine.

def Main():
    global ircsock, Replies
                                # Set up a couple of reply patterns
    pattern1 = '.*:(\w+)\W*%s\W*$' % (botnick)
    pattern2 = '.*:%s\W*(\w+)\W*$' % (botnick)

                                # Create a network socket
    ircsock = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
                                # Connect to server
    ircsock.connect ((server, port))
                                # Authenticate

    ircsock.send ("USER " + uname + " 2 3 " + realname + "\n")

                                # Assign nick
    ircsock.send ("NICK "+ botnick + "\n")
    # Auth to nickserv, uncomment this if needed and change password to the nickserv password
    #ircsock.send ("PRIVMSG NickServ :identify password \n")
    JoinChan (channel)          # Join channel

    while true:                 # Main loop
                                # Receive data from server
        ircmsg = ircsock.recv (bufsize)
                                # Remove newlines
        ircmsg = ircmsg.strip ('\n\r')

        print ircmsg            # Echo the input
                                # Let the bot be a little chatty
                                # Something like "Hello, Bot" ?
        m1 = re.match (pattern1, ircmsg, re.I)
        m2 = re.match (pattern2, ircmsg, re.I)
        if ((m1 == None) and (m2 != None)): m1 = m2

        if (m1 != None):        # Yes
            word = m1.group (1) # Word found
            word = word.lower() # Make word lower case
                                # Print a reply
            if (word in Replies):
                sendmsg (channel, Replies [word])

                                # If the server  pings us,  then we've
                                # got to respond!
        if ircmsg.find ("PING :") != -1:
            ping()

        if ircmsg.find (":!say ") != -1:
            say_split = ircmsg.split ("!say ")
            sendmsg (channel, say_split [1])
            sendmsg (master, "Message sent: " + say_split [1])

        if ircmsg.find ("!commands") != -1:
            sendmsg (channel, "Commands:\n")
            sendmsg (channel, "!say : Say stuff\n")

# welcome message
#        if ircmsg.find ("JOIN " + channel) != -1:
#            sendmsg (channel, "Hello!")

# lulz
        if ircmsg.find ("XD") != -1:
            sendmsg(channel, "Quit saying that! \n")

# change nick
        if ircmsg.find (":!nick ") != -1:
            str_split = ircmsg.split ("!nick ")
            ircsock.send ("NICK "+ str_split [1] + "\n")
            sendmsg (master, "Nick changed to " + str_split [1] + ".")

# send notice
        if ircmsg.find (":!sendnotice ") != -1:
            str_split = ircmsg.split ("!sendnotice ")
            ircsock.send ("NOTICE " + channel + " " + str_split [1] + "\n")
            sendmsg (master, "Notice sent! Notice: " + str_split [1])

# op and deop
        if ircmsg.find ("!op") != -1:
            str_split = ircmsg.split ("!op ")
            ircsock.send ("MODE " + channel + " +o " + str_split [1] + "\n")
            sendmsg (master, "Opped " + str_split [1] + ".")

        if ircmsg.find ("!deop") != -1:
            str_split = ircmsg.split ("!deop ")
            ircsock.send ("MODE " + channel + " -o " + str_split [1] + "\n")
            sendmsg (master, "Deopped " + str_split [1] + ".")
# voice
        if ircmsg.find ("!voice") != -1:
            str_split = ircmsg.split ("!voice ")
            ircsock.send ("MODE " + channel + " +v " + str_split [1] + "\n")
            sendmsg (master, "Voiced " + str_split [1] + ".")

        if ircmsg.find ("!voice") != -1:
            str_split = ircmsg.split ("!voice ")
            ircsock.send ("MODE " + channel + " -v " + str_split [1] + "\n")
            sendmsg (master, "Removed voice on " + str_split [1] + ".")

# set topic
        if ircmsg.find ("!topic") != -1:
            str_split = ircmsg.split ("!topic ")
            ircsock.send ("TOPIC " + channel + " " + str_split [1] + "\n")
            sendmsg (master, "Topic set to: " + str_split [1] + ".")

#---------------------------------------------------------------------
# Main program.

Main()
exit (zero)                     # Exit with success status
#######################################################################################

* [[Plugin development]]

    *Having the bot pull other moduals or scripts from a directory of files
    loading from a cloud server
