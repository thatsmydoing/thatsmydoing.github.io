<!-- 
.. link: 
.. description: 
.. tags: philippine-transit-app, programming, lets-debug
.. date: 2013/07/10 11:30:01
.. title: GTFS Editor
.. slug: gtfs-editor
-->

Link: [https://github.com/conveyal/gtfs-editor](https://github.com/conveyal/gtfs-editor)

**TL;DR** they really meant under development

When I first saw the source of GTFS Editor, I was ecstatic. They used [Play framework](http://playframework.com/)!!! Not only that, they're targeting PostgreSQL as the main database. Those are our favorite tools for building webapps at By Implication. I was a bit sad though, when I saw it was on the 1.x release of Play though. I did have some experience with that release, but not as much compared to 2.x.

Getting it to actually run though, wasn't very pleasant. The initial setup was easy enough. Get [Play 1.2.5](http://www.playframework.com/download), install Postgres with PostGIS, clone the repo and create backing database in Postgres. Some minor additional steps you need are to create the PostGIS extension on the database. The schema is automatically generated and applied by Play so that should be all that's necessary. Wonderful. Then, run play, open a browser, go to [http://localhost:9000](http://localhost:9000), compilation error. Fantastic.

If you don't want to go through the technical details, you can just jump to the [conclusion](#conclusion).

## Let's Debug!

I'll be splitting the next section up into 2 parts. In the first pass, I'll talk about what I did to just get the app to run but I won't try hard to fix any bugs. This generally is what I do when I try to get apps to run. I'll also be dropping enough information so that you can actually figure out what the real problem is. In the second pass, I'll explain what the problems were and how I fixed them.

### First Pass

A thing to note about Play (and one of the reasons it's a lovely Java framework) is that you don't need to do manual compilation. Just edit some source files, refresh your browser and it will automatically do the compilation for you. One less argument for using PHP. It even shows you (in the browser!) the source and which line of code caused the compilation error. So that's what I saw, `Error: type Check already defined`

    :::java
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.METHOD, ElementType.TYPE})
    public @interface Check { // error here
    
        String[] value();
    }

You also know that typical behavior among programmers where your program doesn't compile, but you keep trying to compile it anyway hoping that it will magically just work. That's what I did, and it actually ran. I couldn't really just let this pass, so I decided to try deleting `Check.java`. I got another compilation error, `Error: type Secure already defined`

    :::java
    public class Secure extends Controller { // error here

        @Before(unless={"login", "authenticate", "logout"})

        static void checkAccess() throws Throwable {

At that point, I just decided to just debug it later. It works by just forcing it anyway. So I put `Check.java` back in and proceeded to just refresh until it compiled and ran.

The next problem is a sort of common thing most webapp developers have to solve one way or another. How do you set up the initial admin account? Phrased a different way, how do I login to this thing? The first thing I tried was just add a user into the `account` table directly. One problem though was how to set the password correctly. Plaintext obviously wouldn't work.

Another note regarding Play 1.x, it provides the [secure module](http://www.playframework.com/documentation/1.2.5/secure) which handles logins and keeping state, you simply need to implement the method `boolean authenticate(String username, String password)`. It leaves the actual process of verifying the login to the programmer. This can be exploited by just making the method return `true` and then any login would work. No need to actually set the password. Excellent.

And we're logged in, just in time to encounter a runtime exception. This also works much like compilation errors in Play. It shows a page with the error and the relevant source lines. Now we get, `IndexOutOfBoundsException occured : Index: 0, Size: 0`

    :::java
    if(session.get("agencyId") == null) {
    
        Agency agency = agencies.get(0); // error here
        
        session.put("agencyId", agency.id);
        session.put("agencyName", agency.name);

Apparently, we need to have an agency. That's generally simple enough. You just manually insert an agency into the `agency` table. After that's done, we finally have a view of the actual application. It's very Bootstrap-y, but that's just fine. The workflow though, is not perfectly intuitive, but I'll talk about that some other day.

That's not the end of it though, we still have to fix these bugs. The developer obviously didn't have to put up with this when they were working, so what happened? Also, the log is showing some weird things,

    :::text
    ~        _            _
    ~  _ __ | | __ _ _  _| |
    ~ | '_ \| |/ _' | || |_|
    ~ |  __/|_|\____|\__ (_)
    ~ |_|            |__/
    ~
    ~ play! 1.2.5, http://www.playframework.org
    ~
    ~ Ctrl+C to stop
    ~
    CompilerOracle: exclude jregex/Pretokenizer.next
    Listening for transport dt_socket at address: 8000
    23:32:14,943 INFO  ~ Starting /Users/thomas/Workspace/maps/gtfs-editor
    23:32:14,948 WARN  ~ Declaring modules in application.conf is deprecated. Use dependencies.yml instead (module.secure)
    23:32:14,948 INFO  ~ Module secure is available (/Users/thomas/.root/opt/play-1.2.5/modules/secure)
    23:32:15,830 WARN  ~ You're running Play! in DEV mode
    23:32:15,952 INFO  ~ Listening for HTTP on port 9000 (Waiting a first request to start) ...
    23:32:28,792 ERROR ~
    
    @6f02fa9dd
    Internal Server Error (500) for request GET /
    
    Compilation error (In /app/controllers/Check.java around line 10)
    The file /app/controllers/Check.java could not be compiled. Error raised is : The type Check is already defined
    
    play.exceptions.CompilationException: The type Check is already defined
    	at play.classloading.ApplicationCompiler$2.acceptResult(ApplicationCompiler.java:246)
    	at org.eclipse.jdt.internal.compiler.Compiler.handleInternalException(Compiler.java:672)
    	at org.eclipse.jdt.internal.compiler.Compiler.compile(Compiler.java:516)
    	at play.classloading.ApplicationCompiler.compile(ApplicationCompiler.java:282)
    	at play.classloading.ApplicationClassloader.getAllClasses(ApplicationClassloader.java:426)
    	at play.Play.start(Play.java:516)
    	at play.Play.detectChanges(Play.java:630)
    	at play.Invoker$Invocation.init(Invoker.java:198)
    	at Invocation.HTTP Request(Play!)
    23:32:31,551 INFO  ~ Connected to jdbc:postgresql://127.0.0.1/gtfs_editor
    SLF4J: Class path contains multiple SLF4J bindings.
    SLF4J: Found binding in [jar:file:/Users/thomas/Workspace/maps/gtfs-editor/lib/slf4j-log4j12-1.6.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/Users/thomas/.root/opt/play-1.2.5/framework/lib/slf4j-log4j12-1.6.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
    23:32:32,490 INFO  ~ Initializing HBSpatialExtension
    23:32:32,492 INFO  ~ Attempting to load Hibernate Spatial Provider org.hibernatespatial.postgis.DialectProvider
    23:32:32,494 INFO  ~ Checking for default configuration file.
    23:32:32,496 INFO  ~ No configuration file hibernate-spatial.cfg.xml on the classpath.
    23:32:34,077 INFO  ~ Application 'gtfs-editor' is now started !
    23:32:34,151 INFO  ~ Bootstrapping Database...
    23:32:34,297 DEBUG ~ select count(*) as col_0_0_ from Agency agency0_ limit ?
    play.exceptions.UnexpectedException: Unexpected Error
    	at play.vfs.VirtualFile.contentAsString(VirtualFile.java:180)
    	at play.templates.TemplateLoader.load(TemplateLoader.java:78)
    	at play.test.Fixtures.loadModels(Fixtures.java:174)
    	at jobs.BootstrapDatabase.doJob(BootstrapDatabase.java:57)
    	at play.jobs.Job.doJobWithResult(Job.java:50)
    	at play.jobs.Job.call(Job.java:146)
    	at play.jobs.Job.run(Job.java:132)
    	at play.jobs.JobsPlugin.afterApplicationStart(JobsPlugin.java:116)
    	at play.plugins.PluginCollection.afterApplicationStart(PluginCollection.java:531)
    	at play.Play.start(Play.java:547)
    	at play.Play.detectChanges(Play.java:630)
    	at play.Invoker$Invocation.init(Invoker.java:198)
    	at play.server.PlayHandler$NettyInvocation.init(PlayHandler.java:189)
    	at play.Invoker$Invocation.run(Invoker.java:276)
    	at play.server.PlayHandler$NettyInvocation.run(PlayHandler.java:229)
    	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:439)
    	at java.util.concurrent.FutureTask$Sync.innerRun(FutureTask.java:303)
    	at java.util.concurrent.FutureTask.run(FutureTask.java:138)
    	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$301(ScheduledThreadPoolExecutor.java:98)
    	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:206)
    	at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:895)
    	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:918)
    	at java.lang.Thread.run(Thread.java:680)
    Caused by: play.exceptions.UnexpectedException: Unexpected Error
    	at play.vfs.VirtualFile.inputstream(VirtualFile.java:111)
    	at play.vfs.VirtualFile.contentAsString(VirtualFile.java:178)
    	... 22 more
    Caused by: java.io.FileNotFoundException: /Users/thomas/.root/opt/play-1.2.5/modules/docviewer/app/initial-agencies-data.yml (No such file or directory)
    	at java.io.FileInputStream.open(Native Method)
    	at java.io.FileInputStream.<init>(FileInputStream.java:120)
    	at play.vfs.VirtualFile.inputstream(VirtualFile.java:109)
    	... 23 more
    23:32:34,316 ERROR ~ java.lang.RuntimeException: Cannot load fixture initial-agencies-data.yml: Unexpected Error
    23:32:40,989 DEBUG ~ select account0_.id as id15_, account0_.active as active15_, account0_.admin as admin15_, account0_.agency_id as agency9_15_, account0_.email as email15_, account0_.lastLogin as lastLogin15_, account0_.password as password15_, account0_.passwordChangeToken as password7_15_, account0_.username as username15_ from Account account0_ where account0_.username=? limit ?
    23:32:40,994 DEBUG ~ select count(*) as col_0_0_ from Account account0_ limit ?
    23:32:40,999 DEBUG ~ select nextval ('hibernate_sequence')
    23:32:41,051 DEBUG ~ insert into Account (active, admin, agency_id, email, lastLogin, password, passwordChangeToken, username, id) values (?, ?, ?, ?, ?, ?, ?, ?, ?)
    23:32:41,061 DEBUG ~ select agency0_.id as id24_, agency0_.color as color24_, agency0_.defaultLat as defaultLat24_, agency0_.defaultLon as defaultLon24_, agency0_.defaultRouteType_id as default12_24_, agency0_.gtfsAgencyId as gtfsAgen5_24_, agency0_.lang as lang24_, agency0_.name as name24_, agency0_.phone as phone24_, agency0_.systemMap as systemMap24_, agency0_.timezone as timezone24_, agency0_.url as url24_ from Agency agency0_ order by agency0_.name
    23:32:41,175 ERROR ~
    
    @6f02fa9dg
    Internal Server Error (500) for request GET /
    
    Execution exception (In /app/controllers/Application.java around line 57)
    IndexOutOfBoundsException occured : Index: 0, Size: 0
    
    play.exceptions.JavaExecutionException: Index: 0, Size: 0
    	at play.mvc.ActionInvoker.invoke(ActionInvoker.java:237)
    	at Invocation.HTTP Request(Play!)
    Caused by: java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
    	at java.util.ArrayList.RangeCheck(ArrayList.java:547)
    	at java.util.ArrayList.get(ArrayList.java:322)
    	at controllers.Application.initSession(Application.java:57)
    	at play.mvc.ActionInvoker.invoke(ActionInvoker.java:510)
    	at play.mvc.ActionInvoker.invokeControllerMethod(ActionInvoker.java:484)
    	at play.mvc.ActionInvoker.invokeControllerMethod(ActionInvoker.java:479)
    	at play.mvc.ActionInvoker.handleBefores(ActionInvoker.java:328)
    	at play.mvc.ActionInvoker.invoke(ActionInvoker.java:142)
    	... 1 more

After `23:32:34` is when I get the login page. `23:32:40` is after I've logged in.

### Second Pass

So how did you do? First, the error that `type Check already defined` usually does mean that `Check` was already defined elsewhere. Looking in the app folder though, there was nothing of the sort. It's the only one there that was `Check.java`. But remember the secure module? Modules work by providing source files and Play just compiles them all together. Bingo, `Check.java`. Doing a diff shows nothing was changed. So the solution really was just simply delete `Check.java` and also `Secure.java`. No more compilation errors!

The next question is, how do you get the initial user? There actually is some code that looks like it creates the default admin user,

    :::java
    if(Security.isConnected()) {
        ...
        Account account = Account.find("username = ?", Security.connected()).first();
        ...
        if(account == null && Account.count() == 0) {
            account = new Account("admin", "admin", "admin@test.com", true, null);
            account.save();
        }
        ...
    }

You can actually see this in action at `23:32:41,051` in the log. So what's wrong with all of this? The account creation happened after I've already logged in. In fact, `Security.isConnected()` checks whether the user is already logged in or not. How does this even make sense?

Lastly, we have the problem of the agencies. Just by looking at the log, you can safely say we're missing a file called `initial-agencies-data.yml`. Ok, apparently it's a [fixture](http://www.playframework.com/documentation/1.2.5/test#fixtures) like you would use for testing. It's easy enough to infer what the file's contents should be. We just copy it over from the GTFS data.

But then where do you put the file? If you look at the log, it says `/Users/thomas/.root/opt/play-1.2.5/modules/docviewer/app/initial-agencies-data.yml` but that doesn't look right. That's in the Play distribution directory, probably not somewhere something app-specific should go into. Well, a fixture is used for testing, so maybe the `test/` directory? No, that doesn't work either since we're not running a test.

What I ended up doing was just looking at the sources for `Fixtures.load`. If you follow the stack trace, you end up finding `Play.javaPath` which sort of works like PATH for Fixtures and some other things. So where can we put the file? `app/` and `conf/`. And with that, we're done.

<h3 id="conclusion">Conclusion</h3>

GTFS Editor is very much in development. Just getting it to run was problematic. There also seem to be a lot of missing issues judging from the Github Issues page. If you want to try it out for yourself, I suggest you clone [my branch](https://github.com/thatsmydoing/gtfs-editor) as I've fixed the issues discussed earlier. The default login is `admin:admin`.

Even after getting it to run, it's still not quite usable. Not in the UX sense, but you really can't do much with it. There is no way to import the GTFS data into the webapp. There is something like import from TransitWand but even that is unclear to me. And even if we do get that running as well, we still don't have any data we can play around with. We would need database dumps from the already running tools for these to be of any use right now.