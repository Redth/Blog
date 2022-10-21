# Using the Android Contact Picker with Xamarin.Mobile to Send an SMS in Mono for Android 4.0

First and foremost: Xamarin has been doing an awesome job with their MonoTouch and Mono for Android product line! Huge kudos to the entire team for allowing us some much needed obj-c and java mobile platform escapism!

If you don’t know about Xamarin, they are the company behind products allowing developers to write TRUELY NATIVE applications for Android and iOS in C#! I honestly can’t say enough good things about these products, but you should check them out for yourself: [http://xamarin.com](http://xamarin.com)

Onto the goodies. If it wasn’t enough for us to be able to access native iOS and Android API’s via C#, Xamarin has gone ahead and started building a cross mobile platform set of libraries which aim to help us developers even more in creating sets of code to be reused on all the great mobile platforms. You can check out more about this initiative here: [http://blog.xamarin.com/2011/11/22/introducing-the-xamarin-mobile-api/](http://blog.xamarin.com/2011/11/22/introducing-the-xamarin-mobile-api/)

The latest preview version of Xamarin.Mobile just hit the intertubes, and I was keen to try it out as a replacement to some awful code I had to write to deal with the Android API’s for Contacts. This latest preview now contains Contacts and Location API support for both MonoTouch and Mono for Android. In the sample below, you can see some Android specific code, so I’m not truely writing cross platform code here, but with a bit of abstraction from the UI layer, this would be possible. However, the purpose of this example is more to show you just how bleeping easy it is to work with Contacts in Mono for Android using Xamarin.Mobile!

It’s so simple I don’t even think it needs any explanation past the comments in the code I’ve provided! Enjoy!

PS – A special thanks to @ermau who has been hard at work creating Xamarin.Mobile, and has been super helpful (how cliche of Xamarin folks!) in using it!

    void button_Click(object sender, EventArgs e)
    {
        //Create a new intent for choosing a contact 
        var contactPickerIntent = new Intent(Intent.ActionPick, Android.Provider.ContactsContract.Contacts.ContentUri);
        //Start the contact picker expecting a result 
        // with the resultCode '101'
        StartActivityForResult(contactPickerIntent, 101);
    }
    

    public override void OnActivityResult(int requestCode, Result resultCode, Intent data)
    {
        //See if we are handling the contact picker request code 
        // and that the result code is ok!
        if (requestCode == 101 && resultCode == Result.Ok) { 
            //Ensure we have data returned
            if (data == null || data.Data == null) 
                return;
                
            //Xamarin.Mobile code here 🙂
            var addressBook = new Xamarin.Contacts.AddressBook(this);
            
            //Note: This is important 
            addressBook.PreferContactAggregation = true;
            
            //Load the contact via the android contact id 
            // in the last segment of the Uri returned by the 
            // android contact picker
            var contact = addressBook.Load(data.Data.LastPathSegment);
            
            //Use linq to find a mobile number
            var mobile = (from p in contact.Phones 
                          where p.Type == Xamarin.Contacts.PhoneType.Mobile 
                          select p.Number).FirstOrDefault();
            
            //See if the contact has a mobile number
            if (string.IsNullOrEmpty(mobile)) {
                Toast.MakeText(this, "No Mobile Number for contact!", ToastLength.Short).Show();
                return;
            }
            
            //Send SMS!
            var smsMgr = Android.Telephony.SmsManager.Default; 
            smsMgr.SendTextMessage(mobile, null, "Hello World!", null, null);
        }
    }