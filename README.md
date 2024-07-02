# BasicNumberLocator
This project is a basic number locator using python 
import phonenumbers
from phonenumbers import geocoder as pn_geocoder, timezone, carrier
from tkinter import Tk, Label, Entry, Button, messagebox, Frame, Text, Scrollbar, END
import folium
from folium import Marker
from opencage.geocoder import OpenCageGeocode
import webbrowser

def get_location():
    number = number_entry.get()
    try:
        check_number = phonenumbers.parse(number, "INDIA")
        number_location = pn_geocoder.description_for_number(check_number, "en")
        time_zone = timezone.time_zones_for_number(check_number)
        subscriber = carrier.name_for_number(check_number, "en")

        # Extract state and country
        state_info = pn_geocoder.description_for_number(check_number, "en")
        country_code = check_number.country_code
        country_info = phonenumbers.region_code_for_country_code(country_code)
        state = None
        for i in range(len(state_info)):
            if state_info[i] == ",":
                state = state_info[i+2:]
                break

        # Extract latitude and longitude
        geocoder = OpenCageGeocode("312722d6c0de4bc1a25b2761510c5584")
        results = geocoder.geocode(number_location)
        lat = results[0]['geometry']['lat']
        lng = results[0]['geometry']['lng']

        # Clear previous results
        results_text.config(state='normal')
        results_text.delete(1.0, END)

        # Display new results
        results_text.insert(END, f"Location: {number_location}\nState: {state}\nCountry: {country_info}\nTimezone: {time_zone}\nSubscriber: {subscriber}\n")
        results_text.config(state='disabled')

        # Create map centered on state and district
        map_location = folium.Map(location=[lat, lng], zoom_start=9)
        Marker([lat, lng], popup=number_location).add_to(map_location)
        map_location.save("mylocation.html")

        # Open the generated map in the default web browser
        webbrowser.open_new_tab("mylocation.html")
    except phonenumbers.phonenumberutil.NumberParseException:
        messagebox.showerror("Error", "Invalid phone number.")
    except Exception as e:
        messagebox.showerror("Error", str(e))

# Create the GUI window
root = Tk()
root.title("Phone Number Location Finder")
root.geometry('400x400')  # Set the window size

# Styling
root.configure(bg='white')
frame = Frame(root, bg='lightgray')
frame.pack(padx=10, pady=10, fill='both', expand=True)

# Create and place the entry label and widget
number_label = Label(frame, text="Enter phone number:", bg='lightgray', fg='black')
number_label.grid(row=0, column=0, padx=10, pady=5)
number_entry = Entry(frame, width=30)
number_entry.grid(row=0, column=1, padx=10, pady=5)

# Create and place the search button
search_button = Button(frame, text="Search", command=get_location)
search_button.grid(row=1, column=0, columnspan=2, pady=5)

# Text widget to display results
results_text = Text(frame, height=10, width=40, wrap='word')
results_text.grid(row=2, column=0, columnspan=2, padx=10, pady=5)
results_text.config(state='disabled')

# Add scrollbar
scrollbar = Scrollbar(frame, orient='vertical', command=results_text.yview)
scrollbar.grid(row=2, column=2, sticky='ns')
results_text.config(yscrollcommand=scrollbar.set)

root.mainloop()
