# Assignment_intern
In python scrape data from the link

import requests
import pandas as pd
import numpy as np
from bs4 import BeautifulSoup

#function to extract product title
def get_title(soup):
  try:
      #Outer tag Object
      title = soup.find('span', attrs = {'class':'a-size-medium a-color-base a-text-normal'})
      title_value = title.text
      
      #Title as a string value
      title_string = title_value.strip()

  except AttributeError:
      title_string = ""


  return title_string   

#function to extract Product price
def get_price(soup):
    try:
        price = soup.find('span', attrs = {'class':'a-price'}).string.strip()

    except AttributeError:
      try:
            #if there is some deal price
            price = soup.find('span', attrs = {'class':'a-price-whole'}).string.strip()

        except:
            price = ""

    return price

#Function to extract Product Rating
def get_rating(soup):
    try:        
       rating = soup.find("i", attrs = {'class':"a-icon a-icon-star-small a-star-small-4 aok-align-bottom"})
    except AttributeError:
        try:
            rating = soup.find("span", attrs = {'class':"a-icon-alt"})
        except:
            rating = ""

    return rating

#Function to extract Number of User Reviews
def get_review(soup):
    try:
        review_count = soup.find('div', attrs = {'class':"askInlineWidget"}).text.strip()
    except AttributeError:
        review_count = ""

    return review_count
    
# Required Operation--
    if __name__ == '__main__':
    #Add our user agent
    HEADERS = ({'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36','Accept-Language':'en-US, en;q=00.5'})

    #The webpage url
    url = "https://www.amazon.in/s?k=bags&crid=2M096C61O4MLT&qid=1653308124&sprefix=ba%2Caps%2C283&ref=sr_pg_1"

    #HTTP Request
    webpage = requests.get(url,headers = HEADERS)

    #Soup Object Containing all data
    soup = BeautifulSoup(webpage.content,"html.parser")

    #Fetch links as list of Tag Object
    links = soup.find_all('a', attrs = {'class':'a-link-normal s-underline-text s-underline-link-text s-link-style a-text-normal'})

    #Store the link
    links_list = []

    #Loop for extracting links from tag objects
    for link in links:
        links_list.append(link.get('href'))

    d = {"title":[], "price":[], "rating":[],"reviews":[]}

    #Loop for extracting product details from each link
    for link in links_list:
        new_webpage = requests.get("https://www.amazon.com"+link, headers = HEADERS)

        new_soup = BeautifulSoup(new_webpage.content,"html.parser")


     # Function calls to display all necessery product information
        d['title'].append(get_title(new_soup))
        d['price'].append(get_price(new_soup))
        d['rating'].append(get_rating(new_soup))
        d['reviews'].append(get_review(new_soup))

    amazon_df = pd.DataFrame.from_dict(d)
    amazon_df['title'].replace('',np.nan, inplace= True)
    amazon_df = amazon_df.dropna(subset=["title"])
    amazon_df.to_csv("amazon_data.csv",header= True, index = False)

