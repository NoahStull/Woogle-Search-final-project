# Woogle-Search-final-projectWoogle Search
Introduction
Web Search is one of the foundational pieces of the Internet today. Google built a world spanning multi billion dollar business out of it. In our final project, we are going to build our own search engine! We'll get an insight into how search is performed by crawling the web pages of Western's web site.  This project is therefore called "Woogle Search".
An Outline of Search
Search has three pieces:

Crawling the web to index the set of pages you're interested in. 
Building the index that allows us to quickly search. 
Using the index to perform a search. 

Crawling is an exercise in applying recursion, while building and using the index is an exercise in efficient data structures for text search.
Crawling
The first step is to crawl the web. Starting from some URL, we will visit that web page, index the words found there, then follow the links on that page, and recursively crawl the pages pointed to by the links. Indexing means to find all the words on a page, and creating a map so that words map to the list of pages on which the word appears.

The Crawler class specifies how to crawl.

public class Crawler {
    public InvertedIndex visit(String link, String hostPattern, int depth) {
	…
    }
}

The visit method returns an InvertedIndex as a return value. We will talk more about this in the next section. The inverted index is a classic data structure used for search.

The visit method takes a link as the first argument. That is the page the crawler starts to index. We are going to use "https://wwu.edu" as our starting link. 

The second argument is a host pattern. This will limit the search to only URLs with host names matching the pattern. Our pattern is ".*wwu.edu", so we will limit our crawling to only hosts within Western. If there is a link that goes outside wwu.edu, we won't follow it. We will only index pages within the wwu.edu domain.

The third argument is the depth. If it is 0, then only the root web page will be crawled. If the value is 1, then the root page plus the links it points to are crawled. The depth argument refers to how many levels of links to crawl. In our case, 3 levels is plenty.
Inverted Index
An inverted index is a data structure that maps words to sets of Web pages that they occur in. This is in contrast to a forward index, which maps documents to sets of words. Search engines must optimize the retrieval of a query, that is, it must very quickly find documents that a word occurs in.

The InvertedIndex class represents this data structure.

public class InvertedIndex implements Serializable {

    public PageSet lookup(String word) {
        ...
    }

    public void add(String[] words, Page page) {
        …
    }
}

The InvertedIndex class implements the Serializable interface, which means that it can be stored to disk and retrieved later.  The crawler in the previous section builds an inverted index. This is a lengthy operation, so we can't do it everytime we search. Therefore the crawler stores the inverted index to disk. The search function in the next section reads the inverted index from disk, and uses it to perform searches. You don't have to do anything extra -- just declaring that it implements Serializable is enough to make a class have the ability to persist to disk.

There are only two important methods in InvertedIndex. The first is lookup. This looks up a word and returns a PageSet. A PageSet is a set of web pages that the word is found in. We'll cover PageSet in a later section in more detail.

The second function is add. The add method takes an array of words, and the page, and adds the page to the PageSet for each word. Think of it this way: you found all the words on a web page, and now you need to add the current page to the PageSet for each word on the page. 

Figure 1. An Inverted Index maps words on two map pages to where the pages that the words occur in.

The figure above illustrates this. There are two web pages, stan.html and macbeth.html. The inverted index maps words to the pages where the words occur. Since the word "double" occurs in both stan.html and macbeth.html, the page set for "double" includes both web pages.
Page
A Page represents a web page.

public class Page implements Serializable, Comparable<Page> {
    private String link;
    private int rank;
    …
}

This class is Serializable, so it can be stored to disk as part of the InvertedIndex. 

The link is the link to the page, e.g. http://www.edu/admissions.html

When you do a web search, you want to present the most important pages first. Hence the class also implements Comparable, so that it can be sorted. Pages are sorted by their rank. More important pages have higher rank values. In our Search system, a page's rank is the sum of links that refer to it. That is, the more links there are to a web page, the more important it is. This is a simplification of the Google PageRank algorithm, which states:

PageRank works by counting the number and quality of links to a page to determine a rough estimate of how important the website is. The underlying assumption is that more important websites are likely to receive more links from other websites.
PageSet
A PageSet represents a set of pages.

public class PageSet implements Serializable {
    public boolean add(Page page) { … }

    public boolean contains(Page page) { … }

    public PageSet intersect(PageSet other) {  … }

    public Iterator<Page> iterator() { … }
}

The class has functionality for adding a Page to this set, checking whether the set already contains a page, and calculating the intersection of this set with another set. It also has an iterator that allows you to walk over all Pages in the set.

Adding a Page to this set is important when building the inverted index. 

Intersection is an important operation in search. Two different words will result in two different PageSets. You need to find the set of pages which are present in both PageSets. 
Search
The Search functionality is the simplest piece of the puzzle.

You read in the inverted index that was previously stored by the crawler. 
Read the search query from the user.
Use the inverted index to find the PageSets for each word in the query.
Find the intersection of all the PageSets.
Sort the result by the rank of each Page, and show the search result, highest rank first.
Text Cleaner
The crawler visits web pages and creates the index of words to pages. Every word in the index points to a set of Pages that the word occurs in.  

Some words are uppercase and some are lowercase, and others have mixed case. We want to ignore those differences, so we should convert every word to lowercase.

When you read a web page, some words may have non-alphabetic characters attached to them, e.g. ".group", "fizzy-water", "-grammar,", "covid-19". We're going to take a simple approach and remove all non alphabetic characters in the words, so that the four examples become "group", "fizzywater", "grammar" and "covid19" respectively.  

However, some words are not very useful. For example, "a", "the", "is", "which", "on". These are called stop words. We remove stop words when creating the index, since these words occur frequently and don't add anything meaningful to the index.

The last thing we do is stemming.  Stemming replaces words to their root form. For example, "wait", "waits", "waited", and "waiting" have the same root word "wait". "Fund", "funds", "funded", "funding" all have the same root word "fund". 

The TextCleaner class helps clean up the input by applying non-alpha removal, stemming and stop word removal.

public class TextCleaner {
    public String clean(String word) { … }
}

The clean method takes a word and returns the cleaned up word by applying non-alpha removal, stop word removal and stemming. 

The word is always converted to lowercase.
If the word has non alphabetic characters, it removes those characters.
If the word is a stop word, the return value is the empty string "". 
If the word is like "running", then it returns the stem word "run".
Your Tasks
Your tasks are to build the search engine. To help with that, I've given you a few things:

Java files are in the main directory. These hold:
A full implementation of Page you can use with no modifications
Skeleton implementations of the rest of the classes
TextCleaner
PageSet
InvertedIndex
Crawler
Search
I've also given you tests which are in the directory called Test. These are used to test your code.
The Lib directory holds files that you don't have to modify.
The Jsoup library (it's a jar file -- Java archive). This helps you read HTML web pages and get the words and links on the page. More on this later.
The stop_words.txt file that includes stop words in English.
A prebuilt stemmer you can use in PorterStemmer.java. You don't have to understand how it works -- you just have to use it to stem words.
There is a "Makefile", which is used to build your code.  
As you will see below, you can build, run and test your code by typing various flavors of "make" at the command line.
Make ensures that the right jar files and directories are included so you don't have to worry about adding it all in at the command line.
Task 1. TextCleaner (10 points)
Your first task is to get the TextCleaner class working correctly. The TextCleaner class does lowercasing, non-alpha removal, stop word removal and stemming.  In TextCleaner.clean, your task is to:

Implement lowercase transformation. That is, change all characters to lowercase.
Implement non alphabetic character removal. That is, remove anything that is not an alphabetic character. Hint: check for an isAlphabetic method.
A list of stop words has been provided for you in "Lib/stop_words.txt".  Implement stop word removal. If the input to TextCleaner.clean is a stop word, return the empty string.
Add stemming. You don't have to write your own stemmer! A PorterStemmer class that does stemming using the Porter stemming algorithm has been provided for you.

To use the Porter Stemmer, just create and use one like this:

	Porter stemmer = new PorterStemmer();
String word = "running";
String result = stemmer.stem(word);    // result should be "run"

Test that your TextCleaner works by typing:

	make test-text-cleaner

When all tests pass, you can move on to the next task.
Task 2. PageSet (10 points)
Implement the PageSet class. You need to implement add, contains and intersect, as well as size and iterator. 

You don't have to write the supporting data structure from scratch. Choose an existing Java class in java.util.Collection that gives you the suitable properties you'll need. Build the PageSet using that.

Test your PageSet with:

	make test-page-set

When all tests pass, you can move on to the next task.
Task 3. InvertedIndex (20 points)
Implement the InvertedIndex class. You need to implement add and lookup. Again, you don't have to build the supporting data structures from scratch if you don't want to. You are free to choose a supporting data structure from the standard Java library framework that gives you the properties you need.

Test your PageSet with:

	make test-inverted-index

When all tests pass, you can move on to the next task.
Task 4. Crawl (30 points)
Implement the web crawler. You have to write the visit function that returns an inverted index. 
Recursion
Crawler.visit is a recursive function. The full recursive algorithm is given here for you.

Check if the page was visited before
If the page was never visited before, create a new Page object to represent it. The ranking is set to 1 by the Page constructor. Add the page's base URL (more on this below) to the list of visited pages.
If the page was visited before, call Page.increaseRank to increase its rank by one. You can skip indexing the page since you visited it already, so return directly.
Index the page
Use PageDigest (described in the next section) to read the words on the page
Clean the words using TextCleaner
Add the words and the page to the InvertedIndex
Decrement the depth. If the depth is more than or equal to 0
Get the links on the page with PageDigest (described in the next section)
If a link matches the host pattern, then 
Recursively visit each of the links.
If the link does not match the host pattern, then it links to outside of the domain you want. Skip this link and don't do anything with it.
Parsing Web Pages
I've given you a class called PageDigest that makes parsing HTML web pages easy.  Given a link like "http://wwu.edu", you can read the web page using this code snippet and get all the words in the document.

PageDiest digest = new PageDigest("https://wwu.edu");
String[] words = digest.getWords();   // all the words in the body
            
PageDigest also gives all the links on the page.

ArrayList<String> links = digest.getLinks();

PageDigest is built using the Jsoup library. You can examine the code to see how it works. It should be quite familiar to you if you've been following the lectures.
Checking For Host
You have to decide whether to follow a link or not. The links you follow need to stay within the hostPattern given as an argument to the visit method. 

For our purposes, we want to only index wwu.edu hosts, and not outside links. The pattern you're given is ".*wwu.edu", which is a pattern that says that the host should be "wwu.edu" with zero or more characters in front of it. So hosts like "wwu.edu" or "admissions.wwu.edu" will match the pattern, but "google.com" won't.

To check for matches to the pattern, first import 

	import java.util.regex.Pattern;

Then you can check if a String hostname matches the pattern by

	if (Pattern.matches(hostPattern, hostname)) {
          // match found - the hostname matched the pattern
      }
Pruning Visits
You crawl the web by recursively visiting each page. However, sometimes links lead back to pages that you've already visited.  The easy way to deal with this is to keep a list of URLs that you've already visited as a member variable. That way, if you see a link to somewhere you already visited, you don't have to index it again.

The previously visited list should hold a list of base URLs. That is, the URLs below should all refer to the same visited page, because the base URL is the first one in the list:

	https://wwu.edu/climatechange
https://wwu.edu/climatechange#area1
https://wwu.edu/climatechange?area=1&value=2

The way to do this is to generate a base URL from a link. This code snippet will do that for you:

String link = "https://wwu.edu/climatechange?area=1&value=2";                     
String baseUrl = PageDigest.toBaseUrl(link);   // get the base URL

In the example above, the baseUrl is set to "https://wwu.edu/climatechange".
Ranking
A Page's rank is determined by how many links point to it. When you visit a page:

If the page was never visited before, create a new Page object to represent it. The ranking is set to 1 by the Page constructor. Add the page to the list of visited pages, then continue to index the page.
If the page was visited before, call Page.increaseRank to increase its rank by one, since you just discovered that a link points to it. You can skip indexing the page since you visited it already.
Saving the Inverted Index
The Crawler should save the results in the given file name. Here's a code snippet that can help.

        FileOutputStream fs = new FileOutputStream(filename);
    ObjectOutputStream out = new ObjectOutputStream(fs);
    out.writeObject(invertedIndex);
Running and Testing
You run the crawler by typing

	make crawl

This crawls wwu.edu to a depth of 3, and saves the inverted index result in the file "inverted_index.ser".

You can unit test your crawler by typing

	make test-crawl

When all tests pass, you can move on to the next task.
Task 5. Search (30 points)
Implement Search. Search should read the saved "inverted_index.ser" file. To read the file and reconstruct it as a Java object, use this code snippet:

FileInputStream is = new FileInputStream(filename);
ObjectInputStream in = new ObjectInputStream(is);
InvertedIndex index = (InvertedIndex) in.readObject();

It should then prompt for a search query from the user. It should then:

Use TextCleaner to clean the user's query.
Use InvertedIndex to look up the PageSets for each word 
Find the intersection of all the PageSets
Sort the result so that the higher rank pages come first
Print out the result

You run the search engine by typing

	make search

You can test Search by typing

	make test-search
Finito
If you finished Task 5, congratulations! You have a working search engine of your own! Check it out and play around with it. This simple approach is actually pretty decent in doing web search! 
