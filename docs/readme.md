Option Name:	url_rewrite_program
Replaces:	redirect_program
Requires:	
Default Value:	none
Suggested Config:	

	Specify the location of the executable URL rewriter to use.
	Since they can perform almost any function there isn't one included.

	For each requested URL, the rewriter will receive on line with the format

	  [channel-ID <SP>] URL [<SP> extras]<NL>

	See url_rewrite_extras on how to send "extras" with optional values to
	the helper.
	After processing the request the helper must reply using the following format:

	  [channel-ID <SP>] result [<SP> kv-pairs]

	The result code can be:

	  OK status=30N url="..."
		Redirect the URL to the one supplied in 'url='.
		'status=' is optional and contains the status code to send
		the client in Squids HTTP response. It must be one of the
		HTTP redirect status codes: 301, 302, 303, 307, 308.
		When no status is given Squid will use 302.

	  OK rewrite-url="..."
		Rewrite the URL to the one supplied in 'rewrite-url='.
		The new URL is fetched directly by Squid and returned to
		the client as the response to its request.

	  OK
		When neither of url= and rewrite-url= are sent Squid does
		not change the URL.

	  ERR
		Do not change the URL.

	  BH
		An internal error occurred in the helper, preventing
		a result being identified. The 'message=' key name is
		reserved for delivering a log message.


	In addition to the above kv-pairs Squid also understands the following
	optional kv-pairs received from URL rewriters:
	  clt_conn_tag=TAG
		Associates a TAG with the client TCP connection.
		The TAG is treated as a regular annotation but persists across
		future requests on the client connection rather than just the
		current request. A helper may update the TAG during subsequent
		requests be returning a new kv-pair.

	When using the concurrency= option the protocol is changed by
	introducing a query channel tag in front of the request/response.
	The query channel tag is a number between 0 and concurrency-1.
	This value must be echoed back unchanged to Squid as the first part
	of the response relating to its request.

	WARNING: URL re-writing ability should be avoided whenever possible.
		 Use the URL redirect form of response instead.

	Re-write creates a difference in the state held by the client
	and server. Possibly causing confusion when the server response
	contains snippets of its view state. Embeded URLs, response
	and content Location headers, etc. are not re-written by this
	interface.

	By default, a URL rewriter is not used.
