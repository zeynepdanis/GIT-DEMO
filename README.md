# GIT-DEMO
package com.example.SpringBootRESTAPIWeather.service;


import com.example.SpringBootRESTAPIWeather.model.Weather;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.util.UriTemplate;

import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.math.BigDecimal;
import java.net.URI;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.Collection;
import java.util.Map;


@Service
public class WeatherService {






    private  static final String URL="https://api.openweathermap.org/data/2.5/weather?q={city name}&appid=330a52c650da2c84ca57a6b5d7b1618d&units=metric";

    private String apiKey;

    private final RestTemplate restTemplate;
    private final ObjectMapper objectMapper;
    private Map<String, Object> data;

    public WeatherService(RestTemplateBuilder restTemplateBuilder,ObjectMapper objectMapper){

        this.restTemplate=restTemplateBuilder.build();
        this.objectMapper=objectMapper;
    }

    private final ArrayList<String> cities=new ArrayList<>();
    public ArrayList<String> getCityNames() throws IOException {


        try {

            ObjectMapper mapper = new ObjectMapper();


            JsonNode nodes = mapper.readTree(Paths.get("C:\\Users\\zeynepd\\Desktop\\SpringBootRESTAPIWeather-main\\SpringBootRESTAPIWeather\\src\\main\\resources\\json\\city.list.json").toFile());

            for (JsonNode node : nodes) {


                 cities.add(node.path("name").asText());



            }



        } catch (Exception ex) {
            ex.printStackTrace();
        }

        return cities;


    }
    public Weather getWeather(String city){




            URI url=new UriTemplate(URL).expand(city);
            ResponseEntity<String>response=restTemplate.getForEntity(url,String.class);
            int statusCode = response.getStatusCodeValue();

            if(statusCode==200){
                return convert(city,response);
            }
            else{

                return null;
            }










    }


    private Weather convert(String city,ResponseEntity<String>response){


        try{
            JsonNode root=objectMapper.readTree(response.getBody());
            return new Weather(city,

                    root.path("weather").get(0).path("main").asText(),
                    BigDecimal.valueOf(root.path("main").path("temp").asDouble()),
                     BigDecimal.valueOf(root.path("main").path("feels_like").asDouble()),
                    BigDecimal.valueOf(root.path("wind").path("speed").asDouble()));







        } catch (JsonMappingException e) {
            throw new RuntimeException(e);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

}
