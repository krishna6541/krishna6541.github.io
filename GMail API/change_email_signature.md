pip install --upgrade google-api-python-client google-auth-httplib2 google-auth-oauthlib

from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build
from google.auth.transport.requests import Request
import pickle
import os

# If modifying these scopes, delete the file token.pickle.
SCOPES = ['https://www.googleapis.com/auth/gmail.settings.basic']

def get_gmail_service():
    creds = None
    # The file token.pickle stores the user's access and refresh tokens, and is
    # created automatically when the authorization flow completes for the first
    # time.
    if os.path.exists('token.pickle'):
        with open('token.pickle', 'rb') as token:
            creds = pickle.load(token)
    # If there are no (valid) credentials available, let the user log in.
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        # Save the credentials for the next run
        with open('token.pickle', 'wb') as token:
            pickle.dump(creds, token)

    service = build('gmail', 'v1', credentials=creds)
    return service

def change_gmail_signature(user_id, new_signature):
    service = get_gmail_service()
    
    # Define the signature settings
    signature_settings = {
        'signature': new_signature
    }
    
    # Update the signature
    try:
        service.users().settings().sendAs().patch(
            userId=user_id,
            sendAsEmail=user_id,
            body=signature_settings
        ).execute()
        print("Signature updated successfully.")
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == '__main__':
    user_email = 'your_email@gmail.com'  # Make sure to replace this with your actual email address
    html_signature = """<div>Kind regards,<br>Your Name<br><i>Position</i></div>"""  # Your HTML signature here
    change_gmail_signature(user_email, html_signature)
    
