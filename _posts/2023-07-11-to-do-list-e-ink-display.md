---
title: "To-Do List e-Ink Display"
date: 2023-07-11
layout: default
image: ./images/PXL_20230706_051223183.jpg
categories: jekyll update
excerpt_separator: <!--more-->
---

My partner and I have a to-do list that we maintain in a Google Sheet which contains basic descriptions and due dates of tasks/chores we want to get done. I wrote a very simplistic Python script that will use the Sheets API to access this list and render it on an e-ink display powered by a Raspberry Pi Zero.

<!--more-->

![The finished product](../../../../../images/PXL_20230706_051223183.jpg)

### Google Sheets API Setup

Google's [documentation](https://developers.google.com/sheets/api/quickstart/python) for this is pretty self-explanatory and helped me get this running in short order.

### The code

```python
from inky.auto import auto
from google.oauth2.service_account import Credentials
from googleapiclient.discovery import build
from PIL import Image, ImageDraw, ImageFont

display = auto()

# Load credentials from the service_account.json
creds = Credentials.from_service_account_file('/home/jack/e-ink-to-do-list-f7a78701cd31.json')
service = build('sheets', 'v4', credentials=creds)
sheet = service.spreadsheets()
SPREADSHEET_ID = '<REDACTED>'
RANGE_NAME = 'Sheet1!B4:C50' # The range of cells I care about

# Call the Sheets API
result = sheet.values().get(spreadsheetId=SPREADSHEET_ID, range=RANGE_NAME).execute()
values = result.get('values', [])

# Use a truetype font for rendering the text
fnt = ImageFont.truetype('/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf', 25)

# Calculate maximum width of each column
max_widths = [0] * len(values[0]) # Assumes all rows have the same number of columns
for row in values:
    for i, item in enumerate(row):
        width, _ = fnt.getsize(item)
        max_widths[i] = max(max_widths[i], width)

# Create a new image with white background
img = Image.new('RGB', (800, 480), color = (255, 255, 255))
d = ImageDraw.Draw(img)

# Draw text onto image
y_text = 0
for row in values:
    if y_text > 480: break  # Stop if y position exceeds image height
    x_text = 0
    for i, item in enumerate(row):
        if x_text > 800: break  # Stop if x position exceeds image width
        d.text((x_text, y_text), item, font=fnt, fill=(0, 0, 0))
        x_text += max_widths[i] + 20  # Move x position to next column. Add 20 pixels padding.
    y_text += 30  # Update y position for each new row

# Optionally output the render for debugging
#img.save('output.png')

display.set_image(img)
display.show()
```

### Updating the screen

Unfortunately e-ink displays are notorious for being annoying to update. My solution was to run the update script on a cron job every 20 minutes:

```bash
*/20 * * * * /usr/bin/python3 /home/jack/update-display.py
```

##### Bonus Image

![Back side of the screen](../../../../../images/PXL_20230706_051230560.jpg)

### Future Work

I threw this together in about 45 minutes, so it's not pretty, but it works. I think at some point I'll 3D print a frame for it. The screen has 4 buttons on the side and I could program those to do things like paginate through the list.
