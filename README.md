# Services in Go: from Proof-of-Concept to Production 

It's easy to solve problems in go, but there's a big difference from 
works-on-my-machine to doesn't-break-at-2am-in-production. I'll walk you 
through the build tools, dependency management and deployment techniques 
to get your work into production; then we'll discuss how to do the 
service configuration, coordination and management tools to gracefully 
handle changes in production; and finally the metrics, logging and 
monitoring tools to tackle performance tuning, capacity planning and 
tracking down those bugs that only happen under production load.

This talk was presented at the GolangVan Kickoff meeting on 2014-02-05
17:00 Pacific.

This repo contains the source of the talk, written in [present
format](http://godoc.org/code.google.com/p/go.tools/present). Please
feel free to submit pull requests for typos, errors or improvements.

Slides: http://go-talks.appspot.com/github.com/josephholsten/prod-ready-go-svcs/prod-ready-svcs.slide
Video: https://docs.google.com/file/d/0B4reqmVqk84qQnBSQXlfelc3QTA/preview

## Further Reading

Some of the conversations during and after the talk made me go look up
lots of relevent information. I hope it's useful to you.

### Go Packages

Write your go packages to just work with `go get` and `go build` by
following the guidelines in [Effective
Go](http://golang.org/doc/effective_go.html). More detail is available
in the [Go Language Specification](http://golang.org/ref/spec#Packages).

### Configuration

In depth discussion of [`ARG_MAX`, maximum length of arguments for a new
process](http://www.in-ulm.de/~mascheck/various/argmax/),
and the googler's patch to linux which adds [variable length argument
support](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=b6a2fea39318e43fee84fa7b0b90d68bed92d2ba).

If you want the strict syntax advantages of JSON with the readability
and commentability of YAML, TOML config parsing is available in the
[github.com/bbangert/toml](http://github.com/bbangert/toml) package.
(Thanks to whoever in the audience asked!)

Lua support is available in the
[github.com/stevedonovan/luar](https://github.com/stevedonovan/luar)
package.

### Service Coordination

[Blue Green
Deployments](http://martinfowler.com/bliki/BlueGreenDeployment.html) are
a common deployment pattern that requires orchestration or coordination.

[Etcd](https://github.com/coreos/etcd) is a highly consistant key-value store, with a client avalable in the
[github.com/coreos/go-etcd/etcd](https://github.com/coreos/go-etcd)
package.

[Serf](http://www.serfdom.io) provides fast membership discovery, event
messaging, and (in 0.4) key-value tags.

Oh, and remember how I said I once found a useful go library on
launchpad? That's Canonical's zookeeper client
[`gozk`](https://wiki.ubuntu.com/gozk), available in the
[launchpad.net/gozk](https://launchpad.net/gozk) package. (Thanks [Neno
Lakinski](https://github.com/nenadl)!)


### Signal Handling

GNU libc explains why it's not possible to handle SIGKILL in its [Termination
Signals](https://www.gnu.org/software/libc/manual/html_node/Termination-Signals.html#index-SIGKILL-2899)
documentation. “In fact, if `SIGKILL` fails to terminate a process, that
by itself constitutes an operating system bug which you should report.”

### Process Management

The full explanation of why double-forking/daemonization isn't possible
in go is available at [Issue 227: runtime: support for
daemonize](https://code.google.com/p/go/issues/detail?id=227)

A battle tested and well maintained tool for doing the whole set of
well-behaved daemon behaviour is available in
[daemonize](http://software.clapper.org/daemonize/). (Thanks [Vincent
Janelle](https://twitter.com/randomfrequency)!)

For Python fans, [supervisord](http://supervisord.org) is an excellent
process manager, much like [god](http://godrb.com) or
[bluepill](https://github.com/bluepill-rb/bluepill). Of course, I like
ye olde [monit](http://mmonit.com/monit/).

If you'd like more about my favorite monolithic beast of an init replacement: systemd,
here's the most recent entry in the systemd for system administrators
series: [Socket Activated Internet Services and OS
Containers](http://0pointer.de/blog/projects/socket-activated-containers.html).
It contains references to all previous entries.

Also, since this talk, both
[Debian](http://www.muktware.com/2014/02/debian-technical-committee-votes-systemd-upstart/20780) and
[Upstart](http://www.muktware.com/2014/02/ubuntu-ditch-upstart-switch-systemd/21036) have decided to switch to
systemd. (Thanks [Dan Ballard](http://www.mindstab.net)!)

### Logging

I was wrong in the talk, the `log/syslog` package is totally amenable to the standard
`log` package API! Using the `log` package with a `log/syslog` output is as simple as

    package main

    import (
        "log"
        "log/syslog"
    )

    func main() {
        logger, err := syslog.NewLogger(syslog.LOG_LOCAL0, 0)

        if err != nil {
            log.Fatal("Could not open connection to syslog")
        }

        logger.Print("Hello syslog!")
        logger.Fatal("Goodbye syslog!")
    }


The `nginx` bug for why they do not support syslog (in the open source
version) is [#95 Integrate syslog patch into
modules](http://trac.nginx.org/nginx/ticket/95).

### Profiling

Russ Cox's patch to allow pprof to work on Mac OS X is described in
[Hacking the OS X Kernel for Fun and
Profiles](http://research.swtch.com/macpprof) (Thanks to [Kamil
Kisiel](http://www.kamilkisiel.net)!)

### TODO

* Document opscode's chef-server runit setup

## Thanks

To [Github](http://github.com) for hosting this code. To
[GoDoc](http://godoc.org).

And especially to everyone at Golang Vancouver who pointed me at all
this stuff!
