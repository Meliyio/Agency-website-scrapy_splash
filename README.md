
from urllib.parse import urljoin
import scrapy
from scrapy_splash import SplashRequest

class AGENCY(scrapy.Spider):
    name = 'agency'
    start_urls= ['https://agency.nationwide.com/']

    def start_requests(self):
        for url in self.start_urls:
            yield SplashRequest(url, callback=self.parse, args={'wait':2})

    def parse(self, response, **kwargs):

        links = response.css("a.Directory-listLink::attr(href)").getall()
        for link in links:
            custom_scheme = 'https://agency.nationwide.com/'
            absoulte_url = urljoin(custom_scheme,link)
            yield SplashRequest(absoulte_url, callback=self.parse_page, args={'wait':2})

        return super().parse(response, **kwargs)
    
    def parse_page(self, response, **kwargs):
        links = response.css("a.Directory-listLink::attr(href)").getall()
        for link in links:
            custom_scheme = 'https://agency.nationwide.com/'
            absoulte_url = urljoin(custom_scheme,link)
            yield SplashRequest(absoulte_url, callback=self.parse_page_agency, args={'wait':2})

        
    
    def parse_page_agency(self, response, **kwargs):
        links = response.css("a.Teaser-titleLink::attr(href)").getall()
        for link in links:
            custom_scheme = 'https://agency.nationwide.com/'
            absoulte_url = urljoin(custom_scheme,link)
            yield SplashRequest(absoulte_url, callback=self.parse_page_details, args={'wait':2})

    
    def parse_page_details(self, response, **kwargs):
        name = response.css("span.Core-agencyName::text").get()
        address_line1 = response.css("span.c-address-street-1::text").get()
        address_line2 = response.css("span.c-address-street-2::text").get()
        city = response.css("span.c-address-city::text").get()
        state = response.css("abbr.c-address-state::text").get()
        zip_code = response.css("span.c-address-postal-code::text").get()
        phone = response.css("a.c-phone-number-link c-phone-main-number-link::text").get()
        email = response.css("a.Core-email::attr(href)").get()
        url_address = response.css('a.Core-websiteLink::attr(href)').get()
        yield {
            "name": name.strip() if name else None,
            "address_line1": address_line1.strip() if address_line1 else None,
            "address_line2": address_line2.strip() if address_line2 else None,
            "city": city.strip() if city else None,
            "state": state.strip() if state else None,
            "zip_code": zip_code.strip() if zip_code else None,
            "phone": phone.strip() if phone else None,
            "Email": email.strip() if email else None,
            "url address": url_address.strip() if url_address else None
        }
        
