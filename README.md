# Pipe lab

This lab will introduce you the Unix `|` (pronounced "pipe") operator.
The `|` is similar in spirit to python generators:
it allows us to write programs that use a constant $O(1)$ amount of memory instead of $\Omega(n)$.
Because pipes are efficient, they are commonly used in large scale data analysis pipelines (whence the name).

In this lab, you will use the pipe to do basic analysis of a large twitter dataset.

<img src=img/meme.jpg width=400px />

## Part -1: Setup

You will need a partner to complete this lab.

Create a new github repo.
Then put the contents of this lab into your new repo:
```
$ git clone <your_url>
$ cd <your_repo>
$ git remote add upstream https://github.com/mikeizbicki/lab-pipes-twitter
$ git pull upstream master
$ git push origin master
```

Fork your partner's repo,
then do the work in your partner's repo.
You will have to create a pull request later in the lab.

## Part 0: Measuring the Dataset

The directory `/data/Twitter dataset` contains a large dataset of tweets.
Run the following command to see the size of the dataset
```
$ du -h '/data/Twitter dataset/'
```
The command `du` stands for "disk usage" and `-h` stands for "human readable output".
The `T` in the output stands for [terabytes](https://en.wikipedia.org/wiki/Byte#Multiple-byte_units),
which equals 1000 gigabytes, or $10^9$ bytes.
Over the next few labs and homeworks,
we will practice techniques for working with these large datasets.

> **NOTE:**
> Notice that the path `/data/Twitter dataset` contains a space inside of it.
> That is why the command above has quotation marks around the path.
> Also notice that when I put the path in an inline code segment,
> I do not use quotation marks.
> `'/data/Twitter dataset'` is not the path, because the path does not have quotation marks inside of it.

## Part 1: Counting the Files

The first step in getting familiar with a new dataset is just to count how many files there are.
The standard way to "count" things in the shell is with the `wc` command (which stands for "word count").
Observe that we cannot run the `wc` command on a directory:
```
$ wc '/data/Twitter dataset'
wc: '/data/Twitter dataset': Is a directory
      0       0       0 /data/Twitter dataset
```
Essentially all shell commands take their input on *stdin* (short for "standard input" and pronounced "standard in").
Stdin is normally the keyboard, but the shell can redirect the input to other locations.
For example, we can use heredocs to redirect the input:
```
$ wc <<EOF
> this
> is
> a
> test
> EOF
      4       4      15
```
> **NOTE:**
> In the command above, `$` is the main prompt and indicates that everything typed afterward is a command.
> `>` is a secondary prompt that indicates that this line is still part of the previous command.
> The line
> ```
>      4       4      15
> ```
> has no prompt and so this is the output of the command.

By default, `wc` prints out the number of lines, words, and characters in the input stream.
It is common to add the `-l` flag to only print the number of lines.
Observe:
```
$ wc -l <<EOF
> this
> is
> a
> test
> EOF
4
```

Pipes are a way to connect to programs together.
Recall that the `ls` command will print the contents of a directory,
and so if we count the number of lines, we will count the number of files in the directory.
Connecting ls to pipe like follows gives us the total number of files in our dataset:
```
$ ls '/data/Twitter dataset' | wc -l
```
Importantly, however, not all of these files contain actual data.
For example, there is a file `/data/Twitter dataset/readme.txt` that describes the format of the dataset.

There is another `.txt` file in the dataset folder.
How can we easily find the name of this file?
We can combine the `ls` and `grep` commands like so:
```
$ ls '/data/Twitter dataset' | grep 'txt$'
```
Recall that `grep` takes a regular expression as its argument,
and outputs only those lines that match the regular expression.
In this case, the `$` indicates the end of a line,
and so `grep` will only print those files that end with the letters `txt`.

These two readme files describe how the dataset is structured.
You're not required to view them,
but they explain that:
The data is partitioned into zip files,
where each zip file corresponds to a single day of tweets.
Our next task is to discover how many days worth of data the dataset contains.

Once again, we can answer this question easily with our shell commands.
In this case, we'll need to use `grep` to filter the output of `ls` to keep only files that end in `zip`, then use `wc` to count the results.

> **Exercise 1:**
> Write a one line shell command that returns the number of days worth of data (i.e. the number of zip files) in the twitter dataset.
> Edit the code block below with your command and its output
> ```
> $ ls '/data/Twitter dataset' | grep 'zip$' | wc -l
> 1887
> ```

## Part 2: Counting the Tweets

Our next task is to count how many tweets the dataset has per day,
and once again, we can easily do this with the shell.
The file `/data/Twitter dataset/geoTwitter20-01-01.zip` contains the tweets sent on 01 January 2020,
and we can use the `unzip` command to extract its contents.
Run the command:
```
$ unzip '/data/Twitter dataset/geoTwitter20-01-01.zip'
```
After a few seconds, you should get an error message that looks something like
```
write error (disk full?).  Continue? (y/n/^C)
```
By default, the `unzip` command unzips the contents of the zip file to your current working folder.
You only have 100MB of disk space allocated,
and the file contains about 6GB of data,
so there's no way for you to save the decompressed contents to the hard drive.
Free up your disk space by using the `ls` command to identify the name of the folder created by the `unzip` command, and the `rm` command to remove this folder.

To work with this zip file, we will need a technique that uses only $O(1)$ disk space.
The answer is to have `unzip` print the contents of the file to the screen rather than saving them to the hard drive.
We can do this with the `-p` flag:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip'
```
If you run the command above, you will see a LOT of information printed to the screen,
and it will take more than an hour to finish.
(All of this runtime is in the screen printing, which is very slow.)
Stop the command by pressing CTRL-C on your keyboard.

> **ASIDE:**
> When you press CTRL-C, you will see the character sequence `^C` printed to the terminal.
> The `^` on the terminal is how the CTRL key is represented,
> and so people will sometimes write `^C` instead of CTRL-C.
> Technically, what's happening when you press CTRL-C is that your terminal is sending a special ASCII character to the shell called the [End of Text (ETX)](https://en.wikipedia.org/wiki/End-of-Text_character).
> This is a non-printable character in the ASCII table that doesn't have a key on the keyboard,
> but the CTRL key lets you send non-printable ASCII characters.
> The shell running on the lambda server is never aware that you've pressed the CTRL key.
>
> This is called [caret notation](https://en.wikipedia.org/wiki/Caret_notation), and the use of carat notation is commonly used in programming circles to make jokes like the following:
> ```
> The AVLTree assignment totally sucks^h^h^h^h^hrules.
> ```
> The CTRL-h command is equivalent to the backspace character.
> (Very old keyboards didn't have a backspace character,
> and programmers had to type CTRL-h to delete a letter.)

For commands with a lot of output, it is common to want to inspect only the first few lines of the output.
The `head` command lets us do just that.
Print only the first line of the zip file with the command
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1
```
The first line of the zip file is rather long, so it will still probably fill up your terminal screen,
but the command will at least run quickly.

The format of this zip file is that each line is a JSON (pronounced like the name "Jason") object which contains all the information about a tweet.

We'll take a detailed look at the contents of the JSON object in the next section, but for now, the important thing is that each line of the zip file contains exactly one tweet.
This lets us count the number of tweets sent on a particular day using our standard pipe tools:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | wc -l
```
Unfortunately, the file is so large that just looping over it and counting the number of lines takes a long time.
For me, it takes about 80 seconds when no one else is on the lambda server.

> **ASIDE:**
> If you let the command above run to completion, you'd see that there are about 4 million (or exactly 3930907) tweets in the file.
> These are ALL of the tweets where users have enabled "geolocation" information.
> About 2% of twitter users enable geolocation, and so this is a snapshot of the 80 million tweets that were sent on 01 January 2020.

> **ASIDE:**
> It is useful to time long running commands,
> and the shell has a built-in command `time` for doing this.
> If you re-write the command above as
> ```
> $ time unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | wc -l
> ```
> then the terminal will print the runtime for you.

Let's assume for a second that the best-case runtime for looping over one of these zip files is 80 seconds.
How long would it take to loop over the entire dataset?
We can use the `|` operator combined with the `echo` and `python3` commands to calculate the total number of hours:
```
$ echo "print(1887*80/60/60)" | python3
41.93333333333333
```
> **NOTE:**
> We could rewrite the above to use a heredoc like:
> ```
> $ python3 <<EOF
> print(1887*80/60/60)
> EOF
> ```
> You will commonly see people use both techniques.

Based on this output, we can predict that it will take about 42 hours just to loop over the dataset!
Because looping has a runtime of $\Theta(n)$,
we can estimate that an algorithm with runtime $\Theta(n^2)$ would take time $42*42=1764$ hours, or 73 days to complete.
Therefore, whatever algorithms we perform on this dataset,
we'll want them to have runtime $O(n)$.

> **NOTE:**
> I was careful with my big-O notation above.
> I switched from $\Theta$ to $O$ because it would be okay to have a runtime that was sub-linear and take less than 42 hours.
> They key idea is just that we don't want anything super-linear since that will result in absurd runtimes.
> Big-O lets us express both of these ideas simultaneously in a compact notation.

## Part 3: Inspecting Tweets

In the last task, we saw how to extract an individual tweet using the `head` command:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1
```
The output, however, is hard to read.
In order to save disk space, whitespace has been removed from the tweet's JSON object.

In this section, we will use python to pretty print the tweets for us.

### 3a: inspect.py

The repo contains a command `pretty_print` that lets us visualize json objects nicely.
Run it:
```
$ echo '{"text": "hi", "user": {"screen_name": "a"}}' | ./pretty_print
{
    "text": "hi",
    "user": {
        "screen_name": "a"
    }
}
```
Observe that this command takes unformatted json as input and then pretty prints in with nice formatting that makes the structure obvious.

Now lets look at the contents of the `pretty_print` command:
```
$ cat pretty_print
#!/usr/bin/env python3
import json, sys
file = sys.stdin
data = file.read()
obj  = json.loads(data)
print(json.dumps(obj, indent=4))
```
Internally, the command is just python!

> **NOTE:**
> It is a *human* convention that python files end in `.py`.
> But *computers* do not care about the names of files,
> just like they do not care about the names of variables.
> What tells the computer to run this command using python instead of a different programming language is the *shebang* `#!` in the first line of the file.

The shell redirection operators like `|` work with *all* programs written in *any* programming language.
When working in python, you don't have to do anything special to make your programs support piping,
and it is very common in data science to combine the shell and python.

Now lets use the pretty print command above to inspect the format of our zip file.
We will pipe the contents into the `pretty_print` function.
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | ./pretty_print
```
The command above should run for about a minute and then raise `MemoryError` :(

The problem is that the `.read()` method loads the entire file into memory (and therefore uses $\Theta(n)$ memory for a file of size $n$).
This exceeds the amount of memory you are allowed to use (you can use "only" 5gb of memory at a time and the uncompressed filesize is 6gb).

In order to work with this large dataset, we need to use an $O(1)$ memory algorithm---and that means generators.
Fortunately, every file in python has a generator method `.readlines()` that yields one line of the file at a time.
Rewrite the `pretty_print` file so that it uses this method:
```
#!/usr/bin/env python3
import json, sys
file = sys.stdin
data = file.readlines()[0]    # only this line changes
obj  = json.loads(data)
print(json.dumps(obj, indent=4))
```
Then run your modified program.
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | ./pretty_print
```
It should actually work!
By using the generator, you've reduced the memory usage to $O(1)$ and now we can actually visualize the file.

But it is still slow.
The `.readlines()` generatror loops over every line in the file,
which means our runtime is still $O(n)$.
How can we reduce that runtime?

It turns out that every file object in python is also directly a generator that does not read the entire file.
Therefore, we can call `next` on the `file` object directly to get the next line.

Modify `pretty_print` to replace the `file.readlines()[0]` line with `next(file)`.
Then rerun the command:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | ./pretty_print
```
The command should finish instantly and without errors.
Now you have used generators to reduce both the memory usage and runtime to constant.

> **TAKEAWAY:**
> Careful use of generators is necessary for analyzing large datasets in python!

> **Exercise 2:**
> Commit your changes to the repo.
> Then issue a pull request to your partners repo.
> For full credit on the lab,
> You must have a commit from another person that has made the correct changes to the `pretty_print` command.

> **NOTE:**
> There are two more exercises below.
> You must commit both of these exercises to your own repo and not to your partners!

### 3b: json.tool

In the previous section we saw how to write our own python code to help visualize the contents of a json file.
But this is a common task, and python already has a built-in module called `json.tool` for pretty printing json.
If you run `python3 -m json.tool`, then the stdin will be pretty printed.
All built-in tools like this are design to be maximally memory and runtime efficient.

The following incantation gets us the prettified output:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1 | python3 -m json.tool
```
Observe that we use the `head -n1` command to extract only a single json object before passing to python in this case.
It is considered "ugly" for one command to do multiple things and good practice to combine tools together.
(Just like it is considered ugly for one python function to do multiple things and it is better to combine together multiple python functions.)

> **ASIDE:**
> Technically the output you're looking at is a *javascript object*.
> (JSON stands for *JavaScript Object Notation*, and *object* is the Javascript word for *dictionary*.)
> Conveniently, however, javascript and python have very similar syntaxes.
> The main differences are that javascript does not support single quotes `'` and that it lowercases `true` and `false`.

Find the `"text"` key of the JSON dictionary.

> **NOTE:**
> Notice that I referred to the key as `"text"` and not `text`.
> The content of the string is `text`,
> but the value of the key is the string literal `"text"`.
> Good programmers pay close attention to these grammatical distinctions,
> and it is very easy in a technical interview to detect bad programs by their inability to make these grammatical distinctions.
> If you want the respect of the programming community,
> you should pay attention to our grammar rules.

This is the actual text of the tweet, and you should see that it is equal to
```
🤘🏾 @ Pappadeaux Seafood Kitchen https://t.co/1oecbjJmP2
```
These tweets are sorted based on the time they were sent,
and so we can see that the very first tweet of 2020 was an ad for a seafood restaurant :/

To see who sent this tweet, look at the `"user"` key.
The value for this key is another dictionary containing all the information about the user.
Inside this dictionary, you'll find another key `"screen_name"`,
which shows that @SeanFalyon sent the tweet.

Recall that all of these tweets are geolocated.
That means they contain the location that the twitter user sent them from.
This information is located in the `"place"` key,
which contains a dictionary describing the place the tweet was sent from.
Inside this dictionary is a value `"full_name"` that contains the name of the city and state that the tweet was sent from.
In this case, the tweet was sent from Houston, TX.

## Part 4: `jq`

The terminal has a useful command `jq` for extracting information out of JSON objects.
(`j` is for JSON, and `q` is for query.)
Run the following sequence of commands:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1 | jq '.["text"]'
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1 | jq '.["user"]["screen_name"]'
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n1 | jq '.["place"]["full_name"]'
```
You should see the values of the keys we discussed in the previous section above.

> **Note:**
> The documentation for the `jq` command is [also hosted on github](https://stedolan.github.io/jq/tutorial/),
>
> The syntax can perform incredibly complex queries,
> but you don't need to know the details of this syntax for this class.
> Hopefully, the simple queries above are "obvious".

Let's see one more example using the `jq` command to process multiple tweets at once.
Recall that the `head -n1` command outputs only the first line of its input and discards everything else.
We can change the number `1` to a larger number to get more lines,
and the `jq` command will then process each of those lines individually.
The following command prints the text of the first 20 tweets sent in 2020:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | head -n20 | jq '.["text"]'
```
Notice that many of these tweets are in non-English languages.
(Only about 30% of tweets are English.)

> **Exercise 3:**
> Write a one line shell command that returns the name of the location that the first 10 tweets were sent from on 1 April 2020.
> Edit the code block below with your command and its output
> ```
> $ unzip -p '/data/Twitter dataset/geoTwitter20-04-01.zip' | head -n10 | jq '.["place"]["full_name"]'
> "San José, Costa Rica"
"Phoenix, AZ"
"Cibitung, Indonesia"
null
"Brasília, Brasil"
null
"São Paulo, Brasil"
"Riyadh, Kingdom of Saudi Arabia"
"Queens, NY"
"Dorsten, Deutschland"
> ```

# Part 5: Interesting queries

We're now ready to perform some rather interesting queries on our dataset.
For example, the following query will list the text of all tweets sent from Claremont, CA on new years day 2020:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-01-01.zip' | grep "Claremont, CA" | jq '.["text"]'
```
> **NOTE:**
> You don't need to use the `jq` command to filter a tweet with the `grep` command because `grep` will check all of the fields in the JSON object.
> (This should be "obvious", and make sure you understand why!)
> The `jq` command is most useful for just changing how things get printed.
> Technically, the command above will also catch tweets that contain the string `Claremont, CA` anywhere in their JSON object.
> This could happen, for example, if a person tweets: "I'm in Claremont, CA right now."
> A more technically correct regex would be `"full_name":"Claremont, CA"`,
> but using this more correct regex wouldn't actually change the results of your answer in this case,
> so I've opted for the simpler regex above.

And the following command will count the number of times that "coronavirus" was mentioned on 01 April 2020:
```
$ unzip -p '/data/Twitter dataset/geoTwitter20-04-01.zip' | grep -i "coronavirus" | wc -l
```
> **NOTE:**
> The `-i` flag to `grep` indicates that the regular expression is case insensitive.
> So this regex will match examples like `coronavirus`, `Coronavirus`, `CORONAVIRUS`, and `CoRoNaViRuS`.
> It will also match lines no matter where in the line the text appears,
so it's fine if the text appears inside of a hashtag.
> Also, I'm not using the `jq` command because I'm just counting the results.


> **Exercise 4:**
> Write a one line shell command that outputs the text of all of the tweets sent from Claremont, CA on 12 March 2020 that mention coronavirus.
> (This was the last day of in-person classes that semester.)
> Edit the code block below with your command and its output
> ```
> $ unzip -p '/data/Twitter dataset/geoTwitter20-03-12.zip' | grep "Claremont, CA" | grep -i "coronavirus" | jq '.["text"]'
> "Huh?!🦦"
"Ready to battle Coronavirus. 😤😤😤 https://t.co/DXrh3suIOH"
"I wanna live Bitch 😂😂😂"
"The class of 2020 started their college career w/ the Trump election and will end it with a pandemic. #coronavirus #COVID2019"
".....like Chili’s?!?"
"You can feel it on the air.\n\nNo not Coronavirus cuz it isn’t airborne.\n\nI’m talking about people very slowly turning on AOC."
"@ProfLBNielsen @pomonacollege Yes. There is an application to stay. Some of our international students cannot go ho… https://t.co/tYfxAujxLt"
"When my coworker was calmly telling me how her son looked up conspiracy theories about Coronavirus https://t.co/ECLv7xsR0z"
"I’ll make this my wallpaper for the whole year idc idc https://t.co/b3EMvzwUhB"
"En Iran dicen que si te frotas aceite en el culo, te curas del coronavirus!!\nDe nada"
"Rare W from the juice. #CoronavirusPandemic"
"Todavía no me he pronunciado porque las cosas han cambiado mucho en estos días. Pero sí, en California estamos tamb… https://t.co/kVNIH3JlkT"
"Keeping everyone safe with some common sense #coronavirus #covid19 #savvy109 @ Los Angeles, California https://t.co/gv4Y1P559Q"
"💀"
"https://t.co/lMnmKHbAEn"
"It was me"
"Gonna need statements and dates affected directly from you guys @STAPLESCenter @theforum plz and thank you"
"Everyone at my work be like"
> ```

## Submission

Upload your repo link to canvas.

The answer to Exercise 2 should be committed by your partner,
and the answer to Exercise 3 and 4 should be committed by you.
I don't care who committed Exercise 1.
