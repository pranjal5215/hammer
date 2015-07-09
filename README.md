# Hammer

Hammer is vitess pools's forked implementation along with a basic wrapper

vitess pool implementation has changed since last fork (https://github.com/vireshas/minimal_vitess_pool); this is  version with reforking,

See original vitess pool implementation [here](http://godoc.org/github.com/youtube/vitess/go)

Uses context while doing a get from resorce pool, would be using context to keep track of orphan goroutines/channels created while making a request to external data sources.

####Get the package:
        go get github.com/goibibo/hammerpool
####Code:
package main

import (
	"fmt"
	"time"
	"hammerpool"
	"golang.org/x/net/context"
	"github.com/garyburd/redigo/redis"
)

var pool *hammerpool.ResourcePool

type RedisStruct struct {
	redis.Conn
}

// Close redis conn
func (rConn *RedisStruct) Close(){
	_ = rConn.Conn.Close()
}

// Specify a factory function to create a connection,
// context and a timeout for connection to be created
func init() {
	factory := func() (hammerpool.Resource, error) {
		cli, err := redis.Dial("tcp", "127.0.0.1:6379")
		cli.Do("SELECT", 1)
		if err != nil{
			fmt.Println("Error in Redis Dial")
		}
		res := &RedisStruct{cli}
		return res, nil
	}
	t := time.Duration(5000*time.Millisecond)
	pool = hammerpool.NewResourcePool(factory, 10, 100, t)
}

// Your instance type for redis
func GetMyRedis() (*RedisStruct) {
	return &RedisStruct{}
}

// Execute, get connection from a pool 
// fetch and return connection to a pool.
func (r *RedisStruct) Execute(cmd string, args ...interface{}) (interface{}, error) {
	ctx := context.TODO()
	conn, err := pool.Get(ctx)
	if err != nil {
		return nil, err
	}
	defer pool.Put(conn)
	return conn.(*RedisStruct).Do(cmd, args...)
}

// How to use redis,
func main(){
	rStruct := GetMyRedis()
	value, _ := redis.String(rStruct.Execute("GET", "a"))
	fmt.Println(value)
	value, _ = redis.String(rStruct.Execute("GET", "key"))
	fmt.Println(value)
}

