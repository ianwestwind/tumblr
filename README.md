Tumblr - Part 1
Tumblr App example with required features
Overview
In this lab, you'll build the first part of an app that will allow users to view Tumblr posts. You'll work in collaborative pairs--Pair Programming--to apply the skills you've learned so far building your Flix App Assignment. Just like the Flix App, you'll create a network request to fetch data from the web, however instead of movies, you'll be getting blog post data from the Tumblr API.

Pair Programming - Working in Collaborative Pairs
The checkpoints below should be implemented as pairs. In pair programming, there are two roles: navigator and driver.

 How to Collaborate in Labs - Pair Programming Video: Check out this short video before getting started with the lab.
Navigator: Makes the decision on what step to do next. Their job is to describe the step using high level language ("Let's print out something when the user is scrolling"). They also have a browser open in case they need to do any research.
Driver: is typing and their role is to translate the high level task into code ("Set the scroll view delegate, implement the didScroll method").
After you finish each checkpoint, switch the Navigator and Driver roles. The person on the right will be the first Navigator.
User Stories - What will our app do?
User can view and scroll through a list of Tumblr photo posts.
Let's Get Building!
Milestone 1: Setup
Setup a custom view controller:
Delete the automatically generated ViewController.swift file and create a custom view controller file named PhotosViewController.
Assign the class of the view controller in storyboard to PhotosViewController.
Milestone 2: Hook up the Tumblr API
Send a test request to the Tumblr API:

Using Google Chrome? Add the Pretty JSON Viewer plug-in to your browser makes it easier to visualize the data you get back from the API (in JSON format).
In a browser, access the Tumblr posts for the blog Humans of New York at https://api.tumblr.com/v2/blog/humansofnewyork.tumblr.com/posts/photo?api_key=Q6vHoaVm5L1u2ZAW1fqv3Jw48gFzYVg9P0vH0VHl3GVy6quoGV
Notice that the response dictionary contains two dictionaries, blog and posts. posts is a giant array of dictionaries where each dictionary represents a post and all of it's associated information using key value pairs.
There are often more arrays and dictionaries nested inside the original post dictionaries, as is the case with the image url: photos -> original_size -> url. This will require us to dig into these nested elements and cast to appropriate types as we go.

Create a property in your PhotosViewController class to store posts

This property will store the data returned from the network request. It's convention to create properties near the top of the view controller class where we create our outlets.

Looking at the sample request in the browser, we see the photos key returns an array of dictionaries so that's what type we will make our property.

We will initialize it as an empty array so we don't have to worry about it ever being nil later on.

// 1.       2.             3.
var posts: [[String: Any]] = []
Create a request to the Tumblr Photo Posts Endpoint for the, Humans of New York blog by adding the following network request snippet to the PhotoViewController's viewDidLoad() method.

Note: For the purposes of this lab, we won't go into a lot of detail about the network request code because it's mainly a lot of repetitive configuration that--for the purposes of this course--won't ever change. Whats important for you to know is:
The url specifies which API and to go to and what set of data to retrieve.
If the network request is successful, it will retrieve the information we want from the API endpoint, parse it, and store it in a variable for us to access.
Printing dataDictionary to the console should look very similar to your test request in your browser.
// Network request snippet
let url = URL(string: "https://api.tumblr.com/v2/blog/humansofnewyork.tumblr.com/posts/photo?api_key=Q6vHoaVm5L1u2ZAW1fqv3Jw48gFzYVg9P0vH0VHl3GVy6quoGV")!
let session = URLSession(configuration: .default, delegate: nil, delegateQueue: OperationQueue.main)
session.configuration.requestCachePolicy = .reloadIgnoringLocalCacheData
let task = session.dataTask(with: url) { (data, response, error) in
   if let error = error {
      print(error.localizedDescription)
   } else if let data = data,
      let dataDictionary = try! JSONSerialization.jsonObject(with: data, options: []) as? [String: Any] {
      print(dataDictionary)

      // TODO: Get the posts and store in posts property

      // TODO: Reload the table view
  }
}
task.resume()
Get the posts and store in your posts property

💡 Network requests are asynchronous tasks, which allows the app to continue to execute code while our network request runs in the background. This is key to having responsive app design, otherwise the app would appear frozen while waiting for lengthy processes like network requests to finish. This also means that our table view will initially build before we get any data back from the network. We will need to make sure we reload the table view data any time we get more data back from the network that we want to display.
Use the respective keys and bracket notation to dig into the nested dictionaries.

Cast the value returned from each key to it's respective type.

// Get the dictionary from the response key
let responseDictionary = dataDictionary["response"] as! [String: Any]
// Store the returned array of dictionaries in our posts property
self.posts = responseDictionary["posts"] as! [[String: Any]]
Milestone 3: Build the Photo Feed
Add and Configure a Table View in PhotosViewController:

You can reference steps 1-4 of the Basic Table View Guide
Note: The numberOfRowsInSection method simply tells the table view how many rows, or cells in this case, to create. How many cells do we want? Well, as many as we have posts. We can get that number by calling the count method on our posts array. So, instead of returning a hard coded number like 5 we will want to return posts.count. This is where you can get into trouble if posts contains nil which is why we initialized posts as an empty array because although empty, it is not nil.
Add and Configure a Custom Table View Cell:

We will want to create a custom table view cell, PhotoCell, so we can get it looking just right. You can reference steps 1-2 of the Create the Custom Cell Guide.
Note: Since we are now using a custom cell, inside our tableView(_:cellForRowAt:) method we will change let cell = UITableViewCell() to...
let cell = tableView.dequeueReusableCell(withIdentifier: "PhotoCell", for: indexPath) as! PhotoCell
Setup the Image View in your Custom Cell:

Each cell will need a single UIImageView. Make sure to create an outlet from the image view to your PhotoCell class and not your PhotosViewController class; after all, we created a the custom cell class to control the properties of our reusable cell. DO NOT name this outlet imageView to avoid colliding with the default imageView property of the UITableViewCell base class.
Get the post that corresponds to a particular cell's row:

NOTE: The tableView(_:cellForRowAt:) method is called each time a cell is made or referenced. Each cell will have a unique indexPath.row, starting at 0 for the first cell, 1 for the second and so on. This makes the indexPath.row very useful to pull out objects from an array at particular index points and then use the information from a particular object to populate the views of our cell.
In the tableView(_:cellForRowAt:) method, pull out a single post from our posts array
let post = posts[indexPath.row]
Get the photos dictionary from the post:

It's possible that we may get a nil value for an element in the photos array, i.e. maybe no photos exist for a given post. We can check to make sure it is not nil before unwrapping. We can check using a shorthand swift syntax called if let
post is a dictionary containing information about the post. We can access the photos array of a post using a key and subscript syntax.
photos contains an array of dictionaries so we will cast as such.
// 1.            // 2.          // 3.
if let photos = post["photos"] as? [[String: Any]] {
     // photos is NOT nil, we can use it!
     // TODO: Get the photo url
}
Get the images url:

💡 This is the url location of the image. We'll use our AlamofireImge helper method to fetch that image once we get the url.
Get the first photo in the photos array
Get the original size dictionary from the photo
Get the url string from the original size dictionary
Create a URL using the urlString
// 1.
let photo = photos[0]
// 2.
let originalSize = photo["original_size"] as! [String: Any]
// 3.
let urlString = originalSize["url"] as! String
// 4.
let url = URL(string: urlString)
Set the image view

We'll be bringing in a 3rd party library to help us display our movie poster image. To do that, we'll use a library manager called CocoaPods. If you haven't already, install CocoaPods on your computer now.

Navigate to your project using the Terminal and create a podfile by running, pod init.

Add pod 'AlamofireImage' to your podfile, this will bring in the AlamofireImage library to your project.

In the Terminal run, pod install. When it's finished installing your pods, you'll need to close your xcodeproj and open the newly created xcworkspace file.

import the AlamofireImage framework to your file. Do this at the top of the file under import UIKit. This will allow the file access to the AlamofireImage framework.

import AlamofireImage
call the AlamofireImage method, af_setImage(withURL:) on your image view, passing in the url where it will retrieve the image.

cell.photoImageView.af_setImage(withURL: url!)
Update the table view to display any new information

Our table view will likely be created before we get our photos data back from the network request. Anytime we have fresh or updated data for our table view to display, we need to call:

Do this inside the network request completion handler, right after we load the data we got back into our posts property.

self.tableView.reloadData()
Gotchas
If your app crashes with the exception: Unknown class PhotosViewController in Interface Builder file, try following the steps in this stackoverflow answer.

Compile Error: "No such module AlamofireImage"

Try cleaning and building your project. Command + Shift + K and Command + B






<img src="http://g.recordit.co/lltLKKfE9e.gif" width=250> <br>
