I have caddy running, kept running via systemd script located in /etc/systemd/system, ref here: https://denbeke.be/blog/servers/running-caddy-server-as-a-service-with-systemd/

Site source code is on github, there is a webhook that calls site/gh-update with a secret key.
Caddy is running with the following directive:

www.jeremywho.com {
    redir https://jeremywho.com{uri}
}

jeremywho.com {
    git {
        repo https://github.com/jeremywho/jeremywho.com.git
        hook /gh-update secret
        path ../../jeremywho.com/
       	then hugo
    }


    root jeremywho.com/public
    log logs/jeremywho.com/jeremywho.com.access.log
    errors logs/jeremywho.com/jeremywho.com.error.log
}