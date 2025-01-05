Reactions

```py
@app.route('/send-discord-reaction', methods=['GET'])
def send_discord_reaction():
    # Extract parameters from the URL query string
    token = request.args.get('token')
    channel_id = request.args.get('channelid')
    message_id = request.args.get('messageid')
    emoji = request.args.get('emoji')  # Emoji can be Unicode or custom emoji ID

    # Validate input parameters
    if not token or not channel_id or not message_id or not emoji:
        return make_response("Missing token, channelid, messageid, or emoji parameter", 400)

    # Prepare the Discord API URL for sending the reaction
    discord_api_url = f"https://discord.com/api/v10/channels/{channel_id}/messages/{message_id}/reactions/{emoji}/@me"

    # Create the headers including the authorization token
    headers = {
        "Authorization": f"Bot {token}",
    }

    # Send the PUT request to Discord API to add the reaction
    try:
        response = requests.put(discord_api_url, headers=headers)
        response.raise_for_status()  # Check for HTTP errors
        return jsonify({"status": "Success", "message": "Reaction added"}), response.status_code
    except requests.RequestException as e:
        return make_response(f'{{"status":"Failed", "error":"{str(e)}"}}', 500)
```

Replies
```py
@app.route('/discord-send-reply', methods=['GET'])
def discord_send_reply():
    data = request.args

    token = data.get('token')
    channel_id = data.get('channelid')
    message_id = data.get('messageid')
    content = data.get('content')

    if not token or not channel_id or not message_id or not content:
        return make_response("Missing token, channelid, messageid, or content parameter", 400)

    # Prepare the Discord API URL
    discord_api_url = f"https://discord.com/api/v10/channels/{channel_id}/messages"

    # Create the headers including the authorization token
    headers = {
        "Authorization": f"Bot {token}",
        "Content-Type": "application/json"
    }

    # Create the JSON payload with a reference to the message ID
    payload = {
        "content": content,
        "message_reference": {
            "message_id": message_id
        }
    }

    print(f"DISCORD - REPLY: (Channel: {channel_id}, Message: {message_id}) Content: {content}")

    # Send the POST request to Discord API
    try:
        response = requests.post(discord_api_url, headers=headers, json=payload)
        response.raise_for_status()  # Check for HTTP errors
        return jsonify({"status": "Success", "message": "Reply sent successfully"}), response.status_code
    except requests.RequestException as e:
        return make_response(f'{{"status":"Failed", "error":"{str(e)}"}}', 500)
```

Delete Message
```py
@app.route('/discord-delete-message', methods=['GET'])
def discord_delete_message():
    # Extract parameters from the JSON body of the request
    data = request.args
    
    token = data.get('token')
    channel_id = data.get('channelid')
    message_id = data.get('messageid')

    if not token or not channel_id or not message_id:
        return make_response("Missing token, channelid, or messageid parameter", 400)

    # Prepare the Discord API URL for deleting the message
    discord_api_url = f"https://discord.com/api/v10/channels/{channel_id}/messages/{message_id}"

    # Create the headers including the authorization token
    headers = {
        "Authorization": f"Bot {token}",
        "Content-Type": "application/json"
    }

    print(f"DISCORD - DELETE: (Channel: {channel_id}, Message: {message_id})")

    # Send the DELETE request to Discord API
    try:
        response = requests.delete(discord_api_url, headers=headers)
        response.raise_for_status()  # Check for HTTP errors
        return jsonify({"status": "Success", "message": f"Message {message_id} deleted successfully"}), response.status_code
    except requests.RequestException as e:
        return make_response(f'{{"status":"Failed", "error":"{str(e)}"}}', 500)
```

Replying to interactions
```py
@app.route('/discord-reply-interaction', methods=['GET'])
def discord_reply_interaction():
    # Extract parameters from the JSON body of the request
    data = request.args
    
    interaction_id = data.get('interaction_id')
    interaction_token = data.get('interaction_token')
    content = data.get('content')
    ephemeral = data.get('ephemeral', False)  # Optional, default to False

    if not interaction_id or not interaction_token or not content:
        return make_response("Missing interaction_id, interaction_token, or content parameter", 400)

    # Prepare the Discord API URL for replying to interactions
    discord_api_url = f"https://discord.com/api/v10/interactions/{interaction_id}/{interaction_token}/callback"

    # Prepare the payload for the interaction reply
    payload = {
        "type": 4,  # Type 4 indicates an immediate response
        "data": {
            "content": content,
            "flags": 64 if ephemeral else 0  # Flags 64 makes the reply ephemeral
        }
    }

    print(f"DISCORD - REPLY INTERACTION: (Interaction ID: {interaction_id}, Content: {content})")

    # Send the POST request to Discord API
    response = requests.post(discord_api_url, json=payload)
    response.raise_for_status()  # Check for HTTP errors
    return jsonify({"status": "Success", "message": "Reply sent successfully"}), response.status_code
```
