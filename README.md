## Akka and Avro Serial Evolution

This is a demonstration of using Avro for schema evolution
against Akka Persistence.

This repo is in tutorial form. That means to check it out and 
start with the first commit, and always refer to the README.

1.  The first commit demonstrates how akka persistence works
    against a journal. The `FactPersistenceSpec` works by 
    sending a command to an actor, killing the actor, and
    then querying the actor for its event history.

    If the actor worked correctly, it received the command
    and persisted an event to the persistence journal, and
    then replayed that event from the journal upon restart.
    
    Run the test and proceed to the second commit.
    
2.  Here we start to demonstrate how the java 
    serializer is not ideal for production purposes.
    This step requires a persistence repository that 
    is separate from the codebase, so install a local
    copy of cassandra@2.1 with default configuration.
    
    Note that we've added a "truthMatrix" field 
    to the class that gets persisted - a tuple of
    Doubles. Perhaps our actor calculates both
    the accuracy and popularity rankings of our
    Statement, and stores the values with the Statement.
    
    Now run the app itself. The Evolution app will 
    instantiate a Fact actor and send a command to 
    it, which will in turn persist to cassandra.
    
    Quit and restart the app while keeping cassandra 
    running. You'll see in the logs that the Fact actor's
    recovery process has recovered the previous Statement, 
    and added another Statement to the history.
    
    Keep cassandra running.  Quit the app and proceed 
    to the third commit.
    
3.  In this commit we have upgraded to scala 2.12,
    the next "major version" of scala. Run the app.
    
    You will see errors. This is because Akka Persistence
    is unable to replay the previous event from 
    cassandra.
    
    It couldn't replay because the default java 
    serializer treats scala 2.12 objects differently
    than scala 2.11 objects - in this case, it had
    trouble with the tuple of Doubles.
    
    Generally, one shouldn't use Tuples for 
    persistence events anyway - case classes are
    better. But the point is that java default 
    serialization is not guaranteed to work across 
    major scala versions, and other class structures
    may have trouble as well.
    
    Quit the app and delete your cassandra keyspace:
    
    `cqlsh> drop keyspace akka`
    
    A better serializer will help us transition across
    major scala versions. Proceed to the fourth commit.
    
