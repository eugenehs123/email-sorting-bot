




import os.path
import base64
import google.auth
import google.auth.transport.requests
import google_auth_oauthlib.flow
import googleapiclient.discovery
import googleapiclient.errors
from google.oauth2.credentials import Credentials

# Connect to Gmail
def authenticate_gmail():
    SCOPES = ['https://www.googleapis.com/auth/gmail.readonly']

    creds = None
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    if not creds or not creds.valid:
        if creds and creds.expired and creds.refresh_token:
            creds.refresh(google.auth.transport.requests.Request())
        else:
            flow = google_auth_oauthlib.flow.InstalledAppFlow.from_client_secrets_file(
                'credentials.json', SCOPES)
            creds = flow.run_local_server(port=0)
        with open('token.json', 'w') as token:
            token.write(creds.to_json())

    service = googleapiclient.discovery.build('gmail', 'v1', credentials=creds)
    return service

# Fetch and sort emails
def fetch_and_sort_emails(service):
    results = service.users().messages().list(userId='me').execute()
    messages = results.get('messages', [])

    for msg in messages:
        msg_id = msg['id']
        message = service.users().messages().get(userId='me', id=msg_id).execute()
        msg_snippet = message['snippet']
        msg_subject = ''
        msg_sender = ''
        for header in message['payload']['headers']:
            if header['name'] == 'Subject':
                msg_subject = header['value']
            if header['name'] == 'From':
                msg_sender = header['value']
        # Apply sorting rules here
        # Move emails to corresponding folders
        # Notify user

if __name__ == '__main__':
    service = authenticate_gmail()
    fetch_and_sort_emails(service)

