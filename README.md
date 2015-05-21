# comply

Comply tests compliance of web and net servers with desired output.

It has three modes. All modes support flag options. Only one mode may be
active at once.

If toml is specified, that toml file is read into the configuration. See
the TOML section below for format

If url is specified, it checks only the specified URL with optional IP
address, status code, and regex. If url does not contain a scheme ("https://",
"http://"), "http://" is prefixed. The IP may be an empty string to indicate
all IP addresses resolved to from the URL's hostname.

If hiera is specified, the listeners from it are extracted and tested.

Usage:

	httpComply [flag] [-hiera="path/to/sets.json"] [-toml="/path/to/comply.toml"] [url] [ip] [code] [regex]

The flags are:

	-filter=""
		if specified, only uses this IP address; may end with "." to
		filter by IPs that start with the filter
	-no10=false
		no RFC1918 addresses
	-timeout="3s"
		connection timeout
	-header="X-Request-Guid"
		in more verbose mode, print this HTTP header
	-v
		verbose output:show successes
	-vv
		more verbose output: show -header, cert details

### URLs
URLs may be specified with various protocols: http, https, tcp,
udp, ip. "4" or "6" may be appended to tcp, udp, and ip (as per
[net/#Dial](http://golang.org/pkg/net/#Dial])). tcp and udp must specify a port,
or default to 0. http and https may specify a port to override the default.

### TOML
The [toml file](https://github.com/toml-lang/toml) has two sections: IPs,
a table of search and replace regexes, and Plan, an array of tables listing
test plans.

The IPs table has keys as regexes and values as lists of replacements. A
"*" specifies to perform DNS resolution. If an address contains something
of the form "(x+y)", it is replaced with the sum of x and y.

The plan table array can specify the label, url and ips (array of
string). text, code (number), and regex may be specified for http[s] URLs
to require matching returns. code defaults to 200.

An example file:

	[IPs]
	  BASEIP = ["87.65.43."]
	  '^(\d+)$' = ["*", "BASEIP$1", "BASEIP($1+64)", "1.2.3.$1"]
	  '^(\d+)INT$' = ["*", "10.0.1.$1", "10.0.2.$1", "BASEIP$1", "BASEIP($1+64)"]
	
	[[plan]]
	  label = "api"
	  url = "a href="http://api.example.com/">http://api.example.com/</a>"
	  ips = ["16", "8.7.6.5"]
	  text = "API for example.com"
	  regex = "some regex"
	
	[[plan]]
	  label = "redirect"
	  url = "a href="https://example.com/redirect">https://example.com/redirect</a>"
	  ips = ["*", "20INT"]
	  code = 301
	
	[[plan]]
	  label = "mail"
	  url = "tcp://mail-host.com:25"
