# endless

Zero downtime restarts for golang HTTP and HTTPS servers.


## Inspiration & Credits

Well... it's what you want right - no need to hook in and out on a loadbalancer or something - just compile, SIGHUP, start new one, finish old requests etc.

There is https://github.com/rcrowley/goagain and i looked at https://fitstar.github.io/falcore/hot_restart.html which looked easier to do, but still some assembly required. I wanted something that's ideally as simple as

    err := endless.ListenAndServe("localhost:4242", mux)

I found the excellent post [Graceful Restart in Golang](http://grisha.org/blog/2014/06/03/graceful-restart-in-golang/) by [Grisha Trubetskoy](https://github.com/grisha) and took his code as a start. So a lot of credit to Grisha!


## Features

- Drop-in replacement for `http.ListenAndServe` and `http.ListenAndServeTLS`
- Signal hooks to execute your own code before or after the listened to signals (SIGHUP, SIGUSR1, SIGUSR2, SIGINT, SIGTERM, SIGTSTP)
- You can start multiple servers from one binary and endless will take care of the different sockets/ports assignments when restarting


## Default Timeouts & MaxHeaderBytes

There are three variables exported by the package that control the values set for `DefaultWriteTimeOut`, `DefaultWriteTimeOut`, and `MaxHeaderBytes` on the inner [`http.Server`](https://golang.org/pkg/net/http/#Server):

	DefaultReadTimeOut    time.Duration
	DefaultWriteTimeOut   time.Duration
	DefaultMaxHeaderBytes int

The endless default behaviour is to use the same defaults defined in `net/http`.

These have impact on endless by potentially not letting the parent process die until all connections are handled/finished.


### Hammer Time

To deal with hanging requests on the parent after restarting endless will *hammer* the parent 60 seconds after recieving the shutdown signal from the forked child process. When hammered still running requests get terminated. This behaviour can be controlled by another exported another variable:

    DefaultHammerTime time.Duration

The default is 60 seconds. When set to `-1` `hammerTime()` is not invoked automatically. You can then hammer the parent manually by sending `SIGUSR2`. This will only hammer the parent if it is already in shutdown mode. So unless the process had received a `SIGTERM`, `SIGSTOP`, or `SIGINT` (manually or by forking) before `SIGUSR2` will be ignored.

If you had hanging requests and the server got hammered you will see a log message like this:

    2015/04/04 13:04:10 [STOP - Hammer Time] Forcefully shutting down parent


## Examples & Documentation

Examples are in [here](https://github.com/fvbock/endless/tree/master/examples)

And there is also [Godoc Documentation](https://godoc.org/github.com/fvbock/endless)


## Limitation: No changing of ports

Currently you cannot restart a server on a different port than the previous version was running on.


## TODOs

- tests
- documentation
- less ugly wrapping of the tls.listener
