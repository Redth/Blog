# How to Auto Wire Up Android Activity Views in Xamarin.Android

One thing that’s occasionally bothered me was how verbose it is in Android to wire up views from a layout, in an activity. I finally was fed up enough to write a simple little helper that does this based on reflection. Consider the following verbosity monstrosity:

    [Activity (Label = "RandomDataActivity")]
    public class RandomDataActivity : Activity
    {
        Button button1;
        Button button2;
        Button button3;
        Button button4;
        TextView textView1;
        TextView textView2;
        TextView textView3;
        TextView textView4;
        TextView textView5;
        TextView textView6;
        TextView textView7;
        TextView textView8;
        TextView textView9;
        ImageView imageView1;
        ImageView imageView2;
        ImageView imageView3;
        ImageView imageView4;
        ImageView imageView5;
        ImageView imageView6;
        
        protected override void OnCreate (Bundle bundle)
        {
            base.OnCreate (bundle);
            
            SetContentView (Resource.Layout.RandomDataLayout);
            
            button1 = FindViewById<Button>(Resource.Id.button1);
            button2 = FindViewById<Button>(Resource.Id.button2);
            button3 = FindViewById<Button>(Resource.Id.button3);
            button4 = FindViewById<Button>(Resource.Id.button4);
            textView1 = FindViewById<TextView>(Resource.Id.textView1);
            textView2 = FindViewById<TextView>(Resource.Id.textView2);
            textView3 = FindViewById<TextView>(Resource.Id.textView3);
            textView4 = FindViewById<TextView>(Resource.Id.textView4);
            textView5 = FindViewById<TextView>(Resource.Id.textView5);
            textView6 = FindViewById<TextView>(Resource.Id.textView6);
            textView7 = FindViewById<TextView>(Resource.Id.textView7);
            textView8 = FindViewById<TextView>(Resource.Id.textView8);
            textView9 = FindViewById<TextView>(Resource.Id.textView9);
            imageView1 = FindViewById<ImageView>(Resource.Id.imageView1); 
            imageView2 = FindViewById<ImageView>(Resource.Id.imageView2);
            imageView3 = FindViewById<ImageView>(Resource.Id.imageView3);
            imageView4 = FindViewById<ImageView>(Resource.Id.imageView4);
            imageView5 = FindViewById<ImageView>(Resource.Id.imageView5);
            imageView6 = FindViewById<ImageView>(Resource.Id.imageView6);
        }
    }
    

Now that’s a lot of code to do something really simple. Plumbing code that’s so simple I’d rather not, and _shouldn’t have to_ waste my time on it! How about this instead:

    [Activity (Label = "RandomDataActivity")]
    public class RandomDataActivity : Activity
    {
        Button button1, button2, button3, button4;
        TextView textView1, textView2, textView3, textView4,
                 textView5, textView6, textView7 textView8,
                 textView9;
        ImageView imageView1, imageView2, imageView3, imageView4,
                  imageView5, imageView6;
                
        protected override void OnCreate (Bundle bundle)
        {
            base.OnCreate (bundle);
            SetContentView (Resource.Layout.RandomDataLayout);
            this.WireUpViews ();
        }
    }
    

Here’s the extension method I created to do this:

    public static class ActivityExtensions
    {
        public static void WireUpViews(this Activity activity)
        {
            //Get all the View fields from the activity
            var members = from m in activity.GetType ().GetFields (BindingFlags.NonPublic | BindingFlags.Instance)
                          where m.FieldType.IsSubclassOf (typeof(View)) select m; 
                          
            if (!members.Any ())
                return;
            
            members.ToList ().ForEach (m => { 
                try {
                    //Find the android identifier with the same name
                    var id = activity.Resources.GetIdentifier(m.Name, "id", activity.PackageName);
                    //Set the activity field's value to the view with that identifier 
                    m.SetValue (activity, activity.FindViewById (id)); 
                } catch (Exception ex) {
                    throw new MissingFieldException ("Failed to wire up the field " + m.Name + " to a View in your layout with a corresponding identifier", ex); 
                }
            });
        }
    }
    

Ok, ok, I know it’s not perfect, but it’s a bit of a time saver, and keeps my code looking a bit neater. In a perfect world I should just be able to use an android identifier as if it were already a declared as a code field. There’s also room for improvement in my extension method, for example, what if you declare a field that’s a View but not in the layout, but you did this intentionally? Well, I don’t want to do _**ALL**_ your work for you 😉

Just wanted to share this little bit with you. Hopefully it sparks some ideas for something greater in your day! Happy Coding!