---
layout: post
category: web micro log
tags: [r]
tagline:
---

[General Transit Feed Specification (GTFS)](http://www.transportnsw.info/en/about/transport-data-program.page#gtfs) data is freely available for Sydney Buses. I thought I would go through and see how easy it is to query and create something meaningful.

Reading the files in was rather trivial and required the work of the very common `eval(parse())` combination:

```r
tables <- c("trips", "stops", "stop_times", "routes")

for(tbl in tables) {
  eval(parse(text=paste0(tbl, " <- read.csv(unz('./sydney_buses_gtfs_static.zip','",
                         tbl, ".txt'), row.names=NULL)")
  ))
}
```

My first attempt made use of `plyr` library, but I ended up finding that slightly messy.

```r
# select one route...
all_routes <- unique(grep("X94$", trips$route_id, perl=TRUE, value=TRUE))
all_trips <- unique(as.character(trips[trips$route_id %in% all_routes, c("trip_id")]))

# get all the trips associated with this route
timetable <- stop_times[stop_times$trip_id %in% all_trips, ]
timetable$arrival_time <- as.POSIXct(strptime(as.character(timetable$arrival_time), "%H:%M:%S"))
timetable$departure_time <- as.POSIXctstrptime(as.character(timetable$departure_time), "%H:%M:%S"))

# group by trip id, calculate the difference between min and max of two stop ids.
library(plyr)

# from 203568, to 200045

tt <- timetable[timetable$stop_id %in% c("203568","200045"),]

trip_times <- ddply(tt,
                    .(trip_id),
                    mutate,
                    time_taken = max(arrival_time) - min(arrival_time))
```

But then I realised that this could be much easier user the `dplyr` library with forward piping. The refactored code had less assignments, although much
more condensed.

```r
trip_times <- filter(stop_times, stop_id %in% c("203568", "200045")) %>% # needs to have both!
  group_by(trip_id) %>%
  mutate(time_taken = max(arrival_time) - min(arrival_time)) %>%
  mutate(dest_id=(min(arrival_time)==arrival_time)*as.numeric(stop_id)) %>%
  filter(time_taken != 0) %>%
  filter(dest_id != 0) %>%
  select(trip_id, arrival_time, time_taken, dest_id) %>%
  inner_join(trips[, c("trip_id", "route_id")], by="trip_id") %>%
  filter(grepl("X9[4-9]", route_id)) %>%
  arrange(dest_id, time_taken, arrival_time)
```

Personally this style of the code (though comments would be much better!) is much cleaner. It follows a train of thought and avoids the cognitive overhead
when managing lots of environmental variables. This is definitely something I would exploit more within R in the future.

You can even easily chain these commands to come up with a graph which is shown below.

```r
stop_times %>%
  filter(stop_id %in% c("203568", "200045")) %>% # needs to have both!
  group_by(trip_id) %>%
  mutate(time_taken = max(arrival_time) - min(arrival_time)) %>%
  mutate(dest_id=(min(arrival_time)==arrival_time)*as.numeric(stop_id)) %>%
  filter(time_taken != 0) %>%
  filter(dest_id != 0) %>%
  select(trip_id, arrival_time, time_taken, dest_id) %>%
  inner_join(trips[, c("trip_id", "route_id")], by="trip_id") %>%
  filter(grepl("X9[4-9]", route_id)) %>%
  arrange(dest_id, time_taken, arrival_time) %>%
  ggplot(aes(arrival_time, time_taken)) + geom_line()
```

![bus-times](https://raw2.github.com/chappers/chappers.github.com/master/img/dplyr/Bus Times.png)

Overall, I can definitely see the advantages of forward piping particularly in data analytics, and hope that more and more people would use this approach to generate clean code.
It is quite scary to even think of the code above to be R code since it resembles little of what you might consider to be "base R".
