import base64
import requests , uuid, json
from typing import Union


class Whatsapp:

    def __init__(self,stack_ip,username,password):
        """
        Initialize the Whatsapp class.
        Parameters:
        - stack_ip (str): The IP address or hostname of the WhatsApp stack.
        - username (str): The username for authentication.
        - password (str): The password for authentication.
        """
        self.STACK_IP = stack_ip
        self.BEARER_TOKEN = self._authToken(username,password)


    def _authToken(self,username,password) -> requests.Response:
        """
        Authenticate the user and retrieve the bearer token.

        Parameters:
        - username (str): The username for authentication.
        - password (str): The password for authentication.

        Returns:
        - str: The bearer token obtained after authentication.
        """
        url = f"https://{self.STACK_IP}:9090/v1/users/login"
        payload = json.dumps({
        "new_password": password
        })

        auth_str = f"{username}:{password}"
        encoded_auth = base64.b64encode(auth_str.encode('utf-8')).decode('utf-8')

        headers = {
            'Content-Type': 'application/json',
            'Authorization': f'Basic {encoded_auth}'
        }
        response = requests.request("POST", url, headers=headers, data=payload,verify=False)
        rs = response.text
        json_data = json.loads(str(rs))
        print(json_data)
        token = json_data["users"][0]["token"]
        return token


    def uploadMedia(self,file_name:str,file_binary:bytes,mime_type:str,return_request:bool=False) -> Union[str, requests.Response]:
        media_id = None
        url = f"https://{self.STACK_IP}:9090/v1/media"
        
        files = [
            ("file",(file_name,file_binary,mime_type))
        ]
        payload={"messaging_product":"whatsapp"}
        headers = {
        'Authorization': 'Bearer '+ self.BEARER_TOKEN
        }

        response = requests.request("POST",url=url,headers=headers,data=payload,files=files)
        if return_request:
            return response
        else:
            response = response.json()
            if "id" in response:
                media_id = response["id"]
            return media_id
        
    def get_media_url(self,media_id:str,get_all_media_details:bool=False) -> str:
        url = f"https://{self.STACK_IP}:9090/v1/media/"+media_id
        payload={}
        headers = {
        'Authorization': 'Bearer '+ self.BEARER_TOKEN
        }
        response       = requests.request("GET", url, headers=headers, data=payload)
        media_details  = response.json()
        if get_all_media_details:
            return media_details
        else:
            media_url      = self.__get_media_file(self.BEARER_TOKEN,media_details)
            return media_url
        
    def __get_media_file(self,bearer_token,media_details):

        url = media_details["url"]

        payload={}
        headers = {
        'Authorization': 'Bearer '+ bearer_token
        }

        response = requests.request("GET", url, headers=headers, data=payload)

        media_content = response.content
        media_url = self.__get_url_from_binary(media_content,media_details)

        return media_url

    def __get_url_from_binary(self,file_as_binary,media_details,output_path:str='') -> str:
        # ts              = int(time.time())
        mime_type       = media_details["mime_type"]        #Ex : image/jpeg
        media_type      = mime_type.split("/")[0]           #Ex : image/jpeg -> image
        extension       = mime_type.split("/")[1]           #Ex : image/jpeg -> jpeg
        gen_uuid        = str(uuid.uuid4())
        # file_as_binary = io.BytesIO(file_as_binary)
        key = "media/"+media_type+"/"+"whatsapp_"+media_type+"_"+gen_uuid+"."+extension
        with open(key,'wb') as file:
            file.write(file_as_binary)
        return key


    def _sendMessage(self,payload:dict) -> requests.Response:
        """
        Send a message based on the provided payload.

        Parameters:
        - payload (dict): The payload containing message details.

        Returns:
        - requests.Response: The response object from the HTTP request.
        """
        url = f"https://{self.STACK_IP}:9090/v1/messages"
        headers = {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer '+ self.BEARER_TOKEN
        }
        data = payload
        if isinstance(payload, dict):
            data = json.dumps(payload)
        resp = requests.post(url=url,headers=headers,data=data,verify=False)
        return resp
    

    def sendTextMessage(self,to:str,message:str,preview_url:bool = False) -> requests.Response:    
        payload = _MessageData(to).textMessage(message,preview_url)
        response = self._sendMessage(payload)
        return response

    def sendImageMessage(self,to:str,image_id_or_url:str,caption:str=None) -> requests.Response:
        payload = _MessageData(to).imageMessage(image_id_or_url,caption)
        response = self._sendMessage(payload)
        return response

    def sendAudioMessage(self,to:str,aduio_id_or_url:str) -> requests.Response:    
        payload = _MessageData(to).audioMessage(aduio_id_or_url)
        response = self._sendMessage(payload)
        return response
    
    def sendVideoMessage(self,to:str,video_id_or_url:str,caption:str=None) -> requests.Response: 
        payload = _MessageData(to).videoMessage(video_id_or_url,caption)
        response = self._sendMessage(payload)
        return response

    def sendDocumentMessage(self,to:str,document_id_or_url:str,filename:str='Document',caption:str=None) -> requests.Response: 
        payload = _MessageData(to).documentMessage(document_id_or_url,caption)
        response = self._sendMessage(payload)
        return response
    
    def sendMediaMessage(self,to:str,media_type:str,media_id_or_url,caption:str=None,file_name="Document") -> requests.Response:  
        payload = _MessageData(to).mediaMessage(media_type,media_id_or_url,caption,file_name)
        response = self._sendMessage(payload)
        return response

class _MessageData:
    def __init__(self,to:str):
        self.to = to 


    def textMessage(self,text:str,preview_url:bool = False) -> dict:
        payload = {
            "to": str(self.to),
            "type": "text",
            "text": {
                "body": text,
                "preview_url": preview_url,
            }
        }
        return payload
    

    def imageMessage(self,image_id_or_url:str,caption:str=None) -> dict:
        handle_type = 'id'
        if 'https' in image_id_or_url:
            handle_type = 'link'
        payload = {
            "to": self.to,
            "type": "image",
            "image": {
                handle_type : image_id_or_url
            }
        }       
        if caption:
            payload['image']['caption'] = caption
    
        return payload


    def audioMessage(self,audio_id_or_url:str) -> dict:
        handle_type = 'id'
        if 'https' in audio_id_or_url:
            handle_type = 'link'
        payload = {
            "to": self.to,
            "type": "audio",
            "audio": {
                handle_type : audio_id_or_url
            }
        }
        
        return payload
    
    def videoMessage(self,video_id_or_url:str,caption:str=None) -> dict:
        handle_type = 'id'
        if 'https' in video_id_or_url:
            handle_type = 'link'
        payload = {
            "to": self.to,
            "type": "video",
            "video": {
                handle_type : video_id_or_url
            }
        }
        if caption:
            payload['video']['caption'] = caption
        
        return payload
    
    def documentMessage(self,document_id_or_url,filename:str='Document',caption:str=None) -> dict:
        handle_type = 'id'
        if 'https' in document_id_or_url:
            handle_type = 'link'
        payload = {
            "to": self.to,
            "type": "document",
            "document": {
                handle_type : document_id_or_url,
                "filename": filename
            }
        }
        if caption:
            payload['document']['caption'] = caption
        
        
        return payload
    
    
    def mediaMessage(self,media_type:str,media_id_or_url,caption:str=None,file_name="Document") -> dict:
        handle_type = 'id'
        if 'https' in media_id_or_url:
            handle_type = 'link'
        media_type = media_type.lower()

        payload = {
            "to": self.to,
            "type": media_type,
            media_type: {
                handle_type: media_id_or_url,

            }
        }
        if caption:
            payload['document']['caption'] = caption     
        return payload
    


