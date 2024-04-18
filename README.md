# MesMSG
using System;
using System.Collections.ObjectModel;
using System.IO;
using SQLite;
using Xamarin.Forms;

namespace MultiplatformMessagingApp
{
    public partial class MainPage : ContentPage
    {
        private ContactManager contactManager = new ContactManager();

        public MainPage()
        {
            InitializeComponent();
            contactManager.LoadContacts();
            ContactsListView.ItemsSource = contactManager.Contacts;
        }

        private void SendMessageButton_Clicked(object sender, EventArgs e)
        {
            string senderName = SenderEntry.Text;
            string receiverName = ReceiverEntry.Text;
            string content = MessageEntry.Text;
            MessageType type = MessageType.Text; // Assuming all messages are text for simplicity
            contactManager.SendMessage(senderName, receiverName, content, type);
            MessagesListView.ItemsSource = contactManager.GetMessagesForContact(receiverName);
        }

        private void VoiceCallButton_Clicked(object sender, EventArgs e)
        {
            string callerName = SenderEntry.Text;
            string receiverName = ReceiverEntry.Text;
            contactManager.MakeCall(callerName, receiverName, CallType.Voice);
            // Implement logic to start a voice call
        }

        private void VideoCallButton_Clicked(object sender, EventArgs e)
        {
            string callerName = SenderEntry.Text;
            string receiverName = ReceiverEntry.Text;
            contactManager.MakeCall(callerName, receiverName, CallType.Video);
            // Implement logic to start a video call
        }
    }

    public class ContactManager
    {
        private SQLiteAsyncConnection database;
        private int pageSize = 20; // Number of contacts to load per page

        public ObservableCollection<Contact> Contacts { get; set; }

        public ContactManager()
        {
            string dbPath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData), "contacts.db3");
            database = new SQLiteAsyncConnection(dbPath);
            database.CreateTableAsync<Contact>().Wait();
            Contacts = new ObservableCollection<Contact>();
        }

        public async void LoadContacts()
        {
            // Load contacts from the database
            int pageCount = 0;
            var contacts = await database.Table<Contact>().Skip(pageCount * pageSize).Take(pageSize).ToListAsync();
            
            while (contacts.Count > 0)
            {
                foreach (var contact in contacts)
                {
                    Contacts.Add(contact);
                }
                
                pageCount++;
                contacts = await database.Table<Contact>().Skip(pageCount * pageSize).Take(pageSize).ToListAsync();
            }
        }

        public async void SaveContact(Contact contact)
        {
            await database.InsertAsync(contact);
            Contacts.Add(contact);
        }

        public void SendMessage(string senderName, string receiverName, string content, MessageType type)
        {
            // Send message logic
        }

        public async Task<List<Message>> GetMessagesForContact(string contactName)
        {
            // Get messages for a contact
            return await database.Table<Message>().Where(m => m.Receiver == contactName).ToListAsync();
        }

        public void MakeCall(string callerName, string receiverName, CallType callType)
        {
            // Initiate a voice or video call
        }
    }

    public enum MessageType
    {
        Text,
        Voice
    }

    public enum CallType
    {
        Voice,
        Video
    }

    public class Contact
    {
        [PrimaryKey, AutoIncrement]
        public int ID { get; set; }
        public string Name { get; set; }
        public string PhoneNumber { get; set; }
    }

    public class Message
    {
        public string Sender { get; set; }
        public string Receiver { get; set; }
        public string Content { get; set; }
        public MessageType Type { get; set; }
    }
}
