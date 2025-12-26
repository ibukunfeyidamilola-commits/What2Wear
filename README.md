# What2Wear
import pandas as pd

#read excel file into pandas where data frame is virtual wardrobe.

virtual_wardrobe = pd.read_excel ("/content/comprehensive_outfits.xlsx")




#......................Updating virtual wardrobe.................................
#Use input function but define a function first.

def update_virtual_wardrobe():
  global virtual_wardrobe

  print ("\nIt is a beautiful day to look gorgeous, welcome to What2wear!\n")
  selection = input ("Do you want to update your virtual wardrobe? (yes/no): ")

  if selection.strip().lower() == "yes":
    action = input ("Do you want to add or delete an outfit? (add/delete): ")

    if action.strip().lower() == "add":
      outfit_name = input ("Outfit name with colour e.g. blue jeans jacket: ")
      category = input ("Category e.g. tops: ")
      gender = input ("Gender e.g. female: ")
      season = input ("Season e.g. Spring: ")
      occasion = input ("Occasion e.g. formal: ")

      new_row = {
          "outfit_name" : outfit_name,
          "category" : category,
          "gender" : gender,
          "season" : season,
          "occasion" : occasion
      }

      #Telling pandas to read the dictionary of new row into a data frame and include it in virtual wardrobe then convert to excel
      virtual_wardrobe = pd.concat ([virtual_wardrobe, pd.DataFrame([new_row])], ignore_index= True)
      virtual_wardrobe.to_excel("/content/comprehensive_outfits.xlsx", index =False)

      print ("\nNew outfit has been successfully added to your virtual wardrobe.")

    elif action.strip().lower() == "delete":
      outfit_name = input ("Enter the outfit name to delete: ")
      virtual_wardrobe = virtual_wardrobe [virtual_wardrobe["outfit_name"] != outfit_name]
      virtual_wardrobe.to_excel ("/content/comprehensive_outfits.xlsx", index=False)

      print ("\nOutfit successfully deleted from your virtual wardrobe.")

    else:
      print ("Invalid: Please type 'add' or 'delete' to proceed.")

  else:
    print("\nYou have decided to proceed without updating your virtual wardrobe!")





#.............................Weather to automatically determine season.................................

import datetime as dt
import requests
import json

#datetime will be used to convert the timestamp to the weather API
#requests would be used to send requests to the API

base_url = "https://api.openweathermap.org/data/2.5/weather?"
api_key = "cf5f3bf3d6d16cf6cb25273dbba1ed53"
city = "Hamburg"

url = f"{base_url}q={city}&appid={api_key}"
response = requests.get(url).json()

print(f"General weather Information: ", response)
#The temperature in the API response is in kelvin so there would be a need to change it to celsius


def kelvin_to_celsius(kelvin):
  celsius = kelvin - 273.15
  return celsius


temp_kelvin = response['main']['temp']
temp_celsius = kelvin_to_celsius(temp_kelvin)
feels_like_kelvin = response['main']['feels_like']
feels_like_celsius = kelvin_to_celsius(feels_like_kelvin)
humidity = response['main']['humidity']
description = response['weather'][0]['description']
sunrise_time = dt.datetime.fromtimestamp(response['sys']['sunrise'] + response['timezone'])
sunset_time = dt.datetime.fromtimestamp(response['sys']['sunrise'] + response['timezone'])

print(f"Temperature in {city}: {temp_celsius: .0f}°C, but feels like {feels_like_celsius: .0f}°C")
#print(f"Temperature in {city} feels like: {feels_like_celsius: .2f}°C")
print(f"Humidity in {city}: {humidity}%")
print(f"General Weather in {city}: {description}")
#print(f"Sun rises in {city} at {sunrise_time} local time.")
#print(f"Sun sets in {city} at {sunset_time} local time.")





#.........................................User input..............................................
def user_info():
  print ("\nLet's find you a suitable outfit for today\n")
  user_gender = input ("What is your gender? (male/female): ").strip().lower()
  user_occasion = input ("What kind of occasion are you attending? (casual/formal/semi-formal): ").strip().lower()
  return user_gender, user_occasion





#.......................................Filter outfits based on info..................................
#define a function and filter excel sheet to select based on input.

def outfit_of_the_day (virtual_wardrobe, temperature, description, gender, occasion):
  filtered = virtual_wardrobe [
      (virtual_wardrobe["gender"].str.strip().str.lower() == gender)&
      (virtual_wardrobe["occasion"].str.strip().str.lower() == occasion)
  ]

#match outfit to season in excel based on weather (temperature information), basically define it.

  if temperature < 10:
    weather_season = "Winter"
  elif 10 <= temperature < 15:
    weather_season = "Autumn"
  elif 15 <= temperature < 25:
    weather_season = "Spring"
  else:
    weather_season = "Summer"

  filtered = filtered [filtered["season"].str.strip().str.lower() == weather_season.lower()]


  if filtered.empty:
    print ("\n Unfortunately, no matching outfit found in your virtual wardrobe based on your selection.")
    return None

  else:
    #suggest outfit of the day but we want items in each category selected after gender and season have been filtered.
    #create a list of categories needed
    categories_needed = ["tops", "bottoms", "overalls", "accessories"]
    outfit_combination = {}

    for cat in categories_needed:
      items = filtered [filtered["category"].str.strip().str.lower() == cat]

      if not items.empty:
        outfit_combination [cat] = items.sample(1).iloc[0]["outfit_name"]

    if not outfit_combination:
      print ("\n Unfortunately, there are no suitable outfit pieces in your virtual wardrobe for the categories needed. Return to update virtual wardrobe.")
      return None

    #Now we can suggest the outfit of the day
    print ("\n Based on your information and the weather,:")
    print (f"Weather: {description}, {temperature:.0f}°C")
    print (f"Season of the year: {weather_season}")
    print ("\n Here is your suggested Outfit of the Day:")

    for cat, item in outfit_combination.items():
        print(f"- {cat.capitalize()}: {item}")

    return outfit_combination





#.........................................New list of external suggestions................................

def external_suggestions():
    answer = input("\n Would you like suggestions for outfits you can buy to add to your virtual wardrobe? (yes/no): ").strip().lower()

    if answer == "yes":
        print("\n Here are some outfit suggestions including brands that sell them:")


        new_outfit_suggestions = [
            "- A black blazer (formal & semi-formal) at CnA",
            "- A comfortable pair of sneakers for casual outings at Zalando or Deichmann",
            "- A warm winter coat if temperature is below 10°C at Zalando",
            "- A denim jacket for Spring weather at Zara",
            "- Classic white and black shirts for all occasions at Zara, H&M, CnA",
            "- Stylish handbag or backpack for accessories at Primark",
            "- Sunglasses for sunny days at Christian Dior",
        ]

        for suggestion in new_outfit_suggestions:
            print(suggestion)

        print("\nHappy shopping! We look forward to getting your entries in your virtual wardrobe after your purchase 💕")

    else:
        print("\nGreat! Enjoy your day making statements in your gorgeous outfit!")


#......................................call all functions together.............................................
#update wardrobe
update_virtual_wardrobe()

#Get user info
gender, occasion = user_info()

#Suggest outfit of the day
outfit_of_the_day(virtual_wardrobe, temp_celsius, description, gender, occasion)

#External suggestions
external_suggestions()


