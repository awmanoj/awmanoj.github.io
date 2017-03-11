---
layout: post
title: Writing good error messages (for developers)
author: Manoj Awasthi
categories: tech
---

Writing good software is an Art, albeit the one which can definitely be learnt. As a developer, most of your time would go in maintaining an application - enhancing it with new features, finding bugs, debugging them to isolate the cause and fixing the issues. This post is mostly associated with the debugging of issues.

Writing good error messages should not be an Art. (this is also the goal of this post!)

<img src="/public/assets/img/look.jpg" />

Following are some general comments and suggestions on writing "good" error messages:

(this is by no means a comprehensive document, neither authoritative - just based on my experience)

### Errors should explain the problem concisely and well

Good error messages provide the proper error message that explain the reason well. Easiest way to do that is to include the error code (e.g. errno in C or err.Error() in golang) in the error message.

Example: 

Instead of: 

```
05/Mar/2017:12:08:54 +0700 err something bad happened. 
```

Use: 

```
05/Mar/2017:12:08:54 +0700 err something bad happened. error opening database postgres://a:b@host/maindb
```

### Errors should provide immediate context 

Good error messages provide details of immediate context. In all likelihood, only one error scenario will hit at a time. So each log (however unimportant it may seem) should contain the context e.g. immediate variables related to error condition, exact error code, anything else that may be important etc. 

TIP: while writing code, imagine if this error is hit then what information would you be immediately seeking without attaching debugger. That information should be logged. 

Example: 

Instead of: 

```
05/Mar/2017:12:08:54 +0700 err parsing integer
```

Use:

```
05/Mar/2017:12:08:54 +0700 err parsing integer: "248BAD"
```

### Errors must be actionable 

Don't log an error if it is not actionable. As an example, if you read a record from a database and you get an error that "the record is not present” - if you can proceed without it - for example, you insert the record then do not log this as an error. If you want to count from your logs number of inserts add an “info” log on inserts or may be a “warn” log on error. 

Judge: if you think you don’t need to act on an error in error log - either make it an info or warn or don’t log it.

### Errors must be logged in single line

Don’t use multiline errors. Error logs should contain one (note, one!) error per line. This is to be able to fully use UNIX commands over streaming (pipe).

### Errors should be logged only at top level

Good errors logs are logged at the highest level and hence there is usually only ONE error per occurrence. E.g. 

```
func DoStuff() {
	… 
	if err := DoStuffHelper(); err != nil {
		// log the error since function itself not returning error
		log.Println(“err”, err)  
		return
	}
	...
} 

func DoStuffHelper() error {
	… 
	if err := Stuff(); err != nil {
		// do not log the error since function returning an error. 
		// responsibility of logging error is of the caller
		return err  
	}
	...
}
```

### Errors should roll up in function calls

If there are functions calling other functions and failure happens deep below then it is OK to add some additional error message explanation at each function level (don't log though - see above - just return more detailed error at each level). It is advisable that in such rollup we retain original error code: 

```
func DoA() {
	if err := DoB(); err != nil {
		log.Println("err", err)
		return
	}
}

func DoB() error {
	if err := DoC(); err != nil {
		return errors.New("failed executing C(), " + err.Error())
	}
}

func DoC() error {
	if err := DoD(); err != nil {
		return errors.New("failed executing D(), " + err.Error())
	}
}

...

```

### Conclusion 

Your error log is your friend in bad times - often one of the darkest hours of your work life (problem in production, panic, downtime etc) so an investment in thinking through and writing error logs well will reap significant rewards in saved time and faster resolutions. 

If you have more suggestions do [tweet to me](https://twitter.com/awmanoj).



