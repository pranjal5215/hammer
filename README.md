# vitess_pool_v2

vitess pool implementation has changed since last fork (https://github.com/vireshas/minimal_vitess_pool); this is  version with reforking,

See original vitess pool implementation [here](http://godoc.org/github.com/youtube/vitess/go)

Uses context while doing a get from resorce pool, would be using context to keep track of orphan goroutines/channels created while making a request to external data sources.
