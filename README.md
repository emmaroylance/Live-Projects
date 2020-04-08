# Live Projects
### Introduction
For a total of four weeks of my time at the tech academy, I worked with my peers in a team developing two different websites. 

The first was an interactive website for managing collections of things related to various hobbies, built using the Django framework. I focused on an app about music, that allowed the user to input their own music and save it to their collection. 

The second was an interactive webiste for managing the content and productions for a theater/acting company, built using ASP .Net MVC and Entity Framework. I worked on both back-end and front-end stories for this project.

This was a great learning opportunity for fixing bugs, cleaning up code, and adding requested features. Over the two week sprint I also had the opportunity to work on some other project management and team programming skills that I'm confident I will use again and again on future projects.

Below are descriptions of the step-by-step stories I worked on, along with code snippets that show my work each project.

## Django Framework
  
### Creating a New App for the Project
I created a new app for the website and registered it within the settings file. 

        INSTALLED_APPS = [
            'django.contrib.admin',
            'django.contrib.auth',
            'django.contrib.contenttypes',
            'django.contrib.sessions',
            'django.contrib.messages',
            'django.contrib.staticfiles',
            'FootyDemo',
            'RecipeNomad',
            'PinTrader',
            'MusicGuru',
            'Blazer_Ball',
            ]
I added function to the views file to render the home page

            def home(request):
                return render(request, 'MusicGuru/musicguru_home.html')
    
I registered the urls for my app and the homepage

            urlpatterns = [
                path('', views.home, name='music'),
            ]
I added basic styling to the home page.

          {% extends 'MusicGuru/musicguru_base.html' %}
          {% load staticfiles %}
          {% block templatecontent %}

          <section>
              <div id="footyPic">
                  <img src="{% static 'images/music_home.jpg' %}">
                  <span class="footy_center"><strong>Welcome to the Music Guru App!</strong><br>
                      A collection manager app for a major music fan! There's a database to manage a
                          collection of songs, an API for top hits, and recent favorites scraped from
                          the Pitchfork.com. Take a look.</span>
              </div>
          </section>
          {% endblock %}

### Creating a Model for My App
I created my model for music and added a migration, make sure to plan out all the categories you want to track for your object. I also included an objects manager for accessing the database.

          from django.db import models

          MUSIC_GENRE = (('Pop', 'Pop'), ('Jazz', 'Jazz'), ('Rap', 'Rap'), ('Country', 'Country'), ('Rock', 'Rock'),
                         ('Classical', 'Classical'), ('Folk', 'Folk'), ('Hip-Hop', 'Hip-Hop'), ('Latin', 'Latin'),
                         ('Electronic', 'Electronic'), ('New Age', 'New Age'), ('Other', 'Other'))

          class Song(models.Model):
              song = models.CharField(max_length=30)
              artist = models.CharField(max_length=20)
              genre = models.CharField(max_length=15, choices=MUSIC_GENRE)
              awards = models.CharField(max_length=50, blank=True)
              num_stars = models.IntegerField()
              year_released = models.PositiveIntegerField(blank=True)

              Songs = models.Manager()

              def __str__(self):
                  return self.song

I created a model form that includes any inputs the user needs to make

            from django.forms import ModelForm
            from .models import Song

            #Create the form class.
            class SongForm(ModelForm):
                class Meta:
                    model = Song
                    fields = '__all__'
        
I added a template to my app folder for creating a new item.

            {% extends 'MusicGuru/musicguru_base.html' %}
            {% load staticfiles %}
            {% block templatecontent %}
            <section>
                <div class="flex-container">
                    <form method="post">
                        {% csrf_token %}
                        <table>
                            {{ form.as_table }} <!-- this only renders the td tags, still need the table tags -->
                        </table>
                        {{ form.non_field_errors }}
                        <button class="primary-bright-button" type="submit"> Add to Collection </button>
                    </form>
                    <hr />
                    <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'listSongs' %}'">Back to Collection</button>
                </div>
            </section>
            {% endblock %}

I added a views function that renders the create page and utilizes the model form to save the collection item to the database.

            def add_song(request):
                form = SongForm(request.POST or None)     # Gets the posted form, if one exists
                if form.is_valid():                         # Checks the form for errors, to make sure it's filled in
                    form.save()                             # Saves the valid form/song to the database

                else:
                    print(form.errors)                      # Prints any errors for the posted form to the terminal
                    form = SongForm()                     # Creates a new blank form
                return render(request, 'MusicGuru/musicguru_create.html', {'form': form})


### Creating an Index Page for My App
I created a new index page to display the information from the database and linked it from my home page

      {% extends 'MusicGuru/musicguru_base.html' %}
      {% load staticfiles %}
      {% block templatecontent %}
      <section >
          <div class="flex-container" id="jerseyCollectionPage">
              <table class="table-striped">
                  <tr>
                      <th class="col-md">Song</th>
                      <th class="col-md">Artist</th>
                      <th class="col-md">Genre</th>
                      <th class="col-md">Awards</th>
                      <th class="col-md">Number of Stars</th>
                      <th class="col-md">Year Released</th>
                  </tr>
                  {% for song in songs %}     <!-- creates a new row for each song in the collection -->
                      <tr>
                          <td class="col-md">{{song.song}}</td>
                          <td class="col-md">{{song.artist}}</td>
                          <td class="col-md">{{song.genre}}</td>
                          <td class="col-md">{{song.awards}}</td>
                          <td class="col-md">{{song.num_stars}}</td>
                          <td class="col-md">{{song.year_released}}</td>
                          <td class="col-md">
                              {% if song.authentic %} &#8730 {% endif %}
                          </td>
                      </tr>
                  {% endfor %}
              </table>
              <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'createSong' %}'">Add to Collection</button>
          </div>
      </section>
      {% endblock %}

I added in a function that gets all the items from the database and sends them to the template

    def index(request):
        get_songs = Song.Songs.all()      #Gets all the current songs from the database
        context = {'songs': get_songs}      #Creates a dictionary object of all the songs for the template
        return render(request, 'MusicGuru/musicguru_index.html', context)



### Creating a Details Page for Each Music Addition
Create a details page that will show the details of any single item from within the database, as selected by the user. Link this to the index page for each item.

I added a details template to the template folder and registered the url pattern 

      path('Collection/<int:pk>/Details/', views.details_song, name='songDetails'),    # view details of a song
                  
                  
I created a views function that finds the single desired instance from the database and sends it to the template

    def details_song(request, pk):
        pk = int(pk)                                #Casts value of pk to an int so it's in the proper form
        song = get_object_or_404(Song, pk=pk)   #Gets single instance of the jersey from the database
        context={'song':song}                   #Creates dictionary object to pass the jersey object
        return render(request,'MusicGuru/musicguru_details.html', context)

I added in links for each element on the index page that will direct to the details page for that item

    <td class="col-md"><a href="{{song.pk}}/Details"><button class="primary-light-button">Details</button></a></td>

### Adding Edit & Delete Functions to My App
Allow for edits and delete functions to be done from the details page or from separate pages. Have confirmation before deleting.

I added an edit page to the templates and registered the url pattern

    path('Collection/<int:pk>/Edit/', views.edit_song, name='songEdit'),             # edit a single song


I used model forms and instances to display the content of a single item from the database

    def edit_song(request, pk):
        pk = int(pk)
        song = Song.Songs.get(pk=pk)                                # Alternate way to get the single song from the database
        form = SongForm(data=request.POST or None, instance=song)   # Creates a form filled in with the details of this song
        if request.method == 'POST':                                # If the form is being posted back with changes
            if form.is_valid():                                     # Check that the form is still valid
                form.save()                                         # Save the changes made to the song's details
                return redirect('songDetails', pk)                  # Redirect to the details page for that song
            else:                                                   # Else for form not being valid
                print(form.errors)
        else:                                                       # Else for request not being Post method
            return render(request,'MusicGuru/MusicGuru_edit.html', {'form':form})
         
I created a function very similar to the edit funciton that redirects to the Delete view. 

    def delete_song(request, pk):
        pk = int(pk)
        song = Song.Songs.get(pk=pk)
        context = {'song': song}            #Sets the recipe to a dictionary item for the template
        if request.method == 'POST':            #If the user posts a form, in this case just a delete button
            song.delete()                     #Deletes the recipe from the database
            return redirect('listSongs')            #Redirects back to the index
        else:
            return render(request, 'musicguru/musicguru_delete.html', context)

I created a views function that sends the information for the single item and saves any changes

    {% extends 'MusicGuru/MusicGuru_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section>
        <div class="flex-container">
            <form method="post">
                {% csrf_token %}
                <table>
                    {{ form.as_table }} <!-- this only renders the td tags, still need the table tags -->
                </table>
                {{ form.non_field_errors }}
                <button class="primary-bright-button" type="submit"> Edit Song </button>
            </form>
            <hr />
            <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'listSongs' %}'">Back to Collection</button>
        </div>
    </section>
    {% endblock %}


I added a delete page to the templates and registered the url pattern

    path('Collection/<int:pk>/Delete/', views.delete_song, name='songDelete')
    
I created a views function that lists the song's information, and checks to see if the user wants to delete the song.

    {% extends 'MusicGuru/MusicGuru_base.html' %}
    {% load staticfiles %}
    {% block templatecontent %}
    <section>
        <div class="flex-container footy">
            <div>Are you sure you want to delete this?</div>
            <table>
                <tr>
                    <th>Song Name</th>
                    <td>{{song.song}}</td>
                </tr>
                <tr>
                    <th>Artist</th>
                    <td>{{song.artist}}</td>
                </tr>
                <tr>
                    <th>Genre</th>
                    <td>{{song.genre}}</td>
                </tr>
                <tr>
                <tr>
                    <th>Awards</th>
                    <td>{{song.awards}}</td>
                </tr>
                <tr>
                    <th>Number of Stars</th>
                    <td>{{song.num_stars}}</td>
                </tr>
                <tr>
                    <th>Year Released</th>
                    <td>{{song.year_released}}</td>
                </tr>
            </table>

            <hr />
            <form method="post">
                {% csrf_token %}
            <button class="primary-bright-button" type="submit"> Delete </button>
            </form>
            <button class="primary-bright-button" type="button" onclick=" location.href='{% url 'listSongs' %}'">Cancel</button>
        </div>
    </section>
    {% endblock %}




## ASP .Net MVC and Entity Framework Project

### Story 1:
"Somehow in all our front end upgrades, the CRUD functionality of the Part Views and Controller has gotten messed up. We need to be able to bring this back up to functional.

Go through the Create, Edit, Index, and Delete pages and make sure that you can create a new object, edit that object, display that object in the index (add-on: check details too), and then delete that object. Fix the bugs you hit as you get to them. Make sure that the Edit page is properly populating with the existing data for your item (don't use the first options when creating your object so you can be sure)."

When I clicked on edit, I noticed that it did not show the “Production” or “Person” that were originally created. I fixed this by replacing the following code in the Edit view:

      @Html.DropDownList("Productions", (IEnumerable<SelectListItem>)ViewData["production"], htmlAttributes: new { @class = "form-control" })
      @Html.ValidationMessageFor(model => model.Production)
      </div>
      </div>
      <div class="form-group">
      @Html.LabelFor(model => model.Person, htmlAttributes: new { @class = "control-label col-md-4 inputLabel" })
      <div class="col-md-10 formBox">
      @Html.DropDownList("CastMembers", (IEnumerable<SelectListItem>)ViewData["person"], htmlAttributes: new { @class = "form-control" })
      @Html.ValidationMessageFor(model => model.Person)
      </div>
      </div>

When I tried to save my changes from the edit page, the program froze and sent me to an error that said one of my variables (currentPart) was null. This was because it was defined by a nullable value. In the end I added this code to my Edit View so that it passes the PartID as part of the form:
        
        @Html.HiddenFor(model => model.PartID)

### Story 2:
"We have a new class called Photo that we will be utilizing to store all the images as byte arrays in a single location.
 This is meant to allow for more reusability of our code that deals with images. 
Before we start switching classes over to using this, we need to create an easy way to create new photo objects from the forms of other controllers.

Create a static method within the Photo controller that accepts an HttpPostedFileBase and a string for the image and Title respectively. This method should return an integer. 
The function should 
•	create a new Photo item from the arguments passed into the function, 
o	The PhotoFile attribute should be the image file converted into a byte array.  How to get a byte array from an httpPostedFileBase 
o	The OriginalHeight and OriginalWidth elements should be calculated from the FileBase.
•	save it to the database
•	return the id of the newly created photo item.

 The Title should be the string coming into the function. 

Test your function by replacing the existing code in the Create Post method of the Photo controller with a call to your method."

This is the function I created for this story:
        
        public static int CreatePhoto(HttpPostedFileBase file, string title)

        {
            var photo1 = new Photo();
            using (ApplicationDbContext db = new ApplicationDbContext())
            {
                photo1.Title = Convert.ToString(title);
                Image image = Image.FromStream(file.InputStream, true, true);
                photo1.OriginalHeight = image.Height;
                photo1.OriginalWidth = image.Width;
                Image image1 = Image.FromStream(file.InputStream, true, true);
                var converter = new ImageConverter();
                photo1.PhotoFile = (byte[])converter.ConvertTo(image1, typeof(byte[]));
                db.Photo.Add(photo1);
                db.SaveChanges();
                return (photo1.PhotoId);
            }            
        }



### Story 3:
"Our current calendar (Under Calendar Events Index) only actually displays a calendar when there is a calendar event in the database. When it does display, there are some style issues with it. For instance, it fills most the screen, and the current date does not show the day number (I'm guessing it's white on white). We want to upgrade this UI/UX.

Modify the calendar styling so it fits a month within a typical screen size (or is responsive to screen size), but can still accomodate 3 events each day, and fits with our color scheme a bit better (ie. use some red in there). Use the FullCalendar documentation to figure out how to render the calendar without there being any content in it. Try to make the calendar looks as sleek as possible. Add some calendar events to this month to see you they display, and make sure they look nice too.
current date does not show the day number."

This is the styling I added to the CSS file:

    .palette-container .fc td.fc-today {
        background-color: darkgray;
    }

    .palette-container .fc-content {
        background-color: red;
        border: red;
    }

    .palette-container .fc-event {
        background-color: red;
        border: red;
    }


    .palette-container .fc {
        /* Ensures the calendar doesn't become too wide on large screens */
        max-width: 1000px;
        /* Lowers calendar*/
        padding-top: 16px;
        /*Centers calendar*/	
        margin: auto;
    }

I added "contentHeight: 550", to the Index view under Calendar Events, and that helped with the height issue.

In the Index view under Calendar Events, there are a couple of lines of code that place the events listed in to an array, and then tries to modify them. I removed this code, as it was no longer needed, and that fixed the issue of the calendar only displaying when there are events in the database. 


### Story 4:
"Currently, photos in the production page are stored in our database as byte arrays. The problem with this is that the dimensions of the original image that is uploaded are not stored as well. We have created a class to store all the photos along with details about the photos(title and original dimensions). You need to do the following:
1.	Modify the ProductionPhotos model so that photos are stored as integer rather than a byte array
2.	Modify the create method in the ProductionPhotos Controller to use the photo class that we have created
3.	Modify the ProductionPhotos views to then display the photos we have stored using the photo class"

I modified the ProductionPhotos model so that photos are stored as integer rather than a byte array, and renamed it to a more accurate description:

        public int PhotoId { get; set; }    
        
I modified the create method in the ProdutionPhotos Controller to use the photo class that we have created by changing this code:
        
        public ActionResult Create([Bind(Include = "Title,Description,Production")] ProductionPhotos productionPhotos, HttpPostedFileBase file)
        {
            int productionID = Convert.ToInt32(Request.Form["Productions"]);
            
            byte[] photo = ImageUploadController.ImageBytes(file, out string _64);
            productionPhotos.Photo = photo;

            if (ModelState.IsValid)
            {
                var production = db.Productions.Find(productionID);

                productionPhotos.Production = production;
                db.ProductionPhotos.Add(productionPhotos);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(productionPhotos);
        }



to this code:

        public ActionResult Create([Bind(Include = "Title,Description,Production")] ProductionPhotos productionPhotos, HttpPostedFileBase file)
        {
            int productionID = Convert.ToInt32(Request.Form["Productions"]);

            productionPhotos.PhotoId = PhotoController.CreatePhoto(file, productionPhotos.Title);

            if (ModelState.IsValid)
            {
                var production = db.Productions.Find(productionID);

                productionPhotos.Production = production;
                db.ProductionPhotos.Add(productionPhotos);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(productionPhotos);
        }
        
        
I modified the ProductionPhotos views to then display the photos we have stored using the photo class. To do this, I had to change every reference to "Photo" within the views to "PhotoId". 
For the Index, Details, Delete and Edit views, I changed the img src to reflect these changes. The code changed from 

            <td class="td-styling">             
                @{
                    string img = "";
                    if (item.Photo != null)
                    {
                        byte[] thumbBytes = ImageUploadController.ImageThumbnail(item.Photo, 100, 100);
                        var base64 = System.Convert.ToBase64String(thumbBytes);
                        img = String.Format("data:image/png;base64,{0}", base64);

                    }
                }
                
                <img src="@img" class="thumbnail_size"/>
                
            </td>
            
to

            <td class="td-styling">             
                <img src='@Url.Action("DisplayPhoto", "Photo", new { id = item.PhotoId })' />
            </td>
            
The Create view does not currently show any images. The only edit I made from that view, was to change this block of code 

                    <div class="col-md-10 formBox">
                        @Html.TextBoxFor(model => model.Photo, new { type = "file", Name = "file" , Class = "fileSelect " })
                        @Html.ValidationMessageFor(model => model.Photo, "", new { @class = "text-danger" })
                    </div>
to

                    <div class="col-md-10 formBox">
                        <input type="file" name="file" required />
                        @Html.ValidationMessageFor(model => model.PhotoId, "", new { @class = "text-danger" })
                    </div>
This change helps with a problem that comes with the change from a byte array to an integer.                     


<input type="file" name="file" required />

### Other Skills Learned
* Working with a group of developers to identify front and back end bugs to the improve usability of an application
* Improving project flow by communicating about who needs to check out which files for their current story
* Learning new efficiencies from other developers by observing their workflow and asking questions
* Using Azure DevOps to update progress, and maintain clean version control






