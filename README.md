## Write Up
Our project began once again with a desire to utilize and visualize the Our World In Data
Energy usage data as a method to explore the how behind how the World, and each of its
countries get their electricity as an analogy for how CO2 producing fossil fuels, or lack thereof
powers both daily life and industry. Notably though, this is on an absolute scale, which may
seem like another variation of a population map, but there are some notable exceptions and
differences. Firstly, while it is true that countries with more population need more energy, it
doesn’t change the quantity of energy being used, and fuel being burned to accommodate. For
example, the US has a higher Fossil Fuel per capita usage of electricity, it’s also true that in the
end, China is flat out burning more to sustain their larger population, and in this case, per capita
usage does not describe the full picture. To mitigate this though, there are also revealing trends
when the data is broken up into specific sections of energy usage per generation source, and a
bar graph to show the aggregate. The first visualization, Fossil Fuel, is also an aggregated data
source, so it’s important to see the specifics and individual trends (China Coal, US Natural Gas,
Saudi Arabia Oil, etc.) to see the hows rather than just viewing the aggregated data on an
absolute scale.

To this end, utilizing a Chloropleth seemed like a far more intuitive option than utilizing
a more basic form of interaction, such as a line or bar graph to display energy usage. The scale of
countries in the world is dense, so utilizing something with potentially more specificity like a
line or bar chart really only serves to effectively display the extreme ends of the spectrum, as the
scale of power generation for smaller countries disappear into either illegible text, or a blur of
colors. In contrast, a color range spread across a world map easily conveys the where and how
much of each country in the world, as there is now a geographic display where each country can
have their own individual space rather than needing to share one axis as would be in the case
with a more graphical encoding technique. The ability to select individual year also allows for
more visuals to be encoded within the same space, as through cycling through the years, or
checking specific years, the choropleth also serves as a way for an end user to visualize change
over time for each energy generation source.

A choropleth also serves as a useful geometric way for a user to hover and see the data
they want, and serves as its own form of interactivity in that way. Rather than needing to search
for the data in a text box, each country is now visible geographically, and thus only requires
knowledge of a world map. This also comes with the benefit of being able to compare the

distribution of energy usage, both to their neighboring countries, and also to any other countries
the end user wants far easier than needing to multi select countries of interest.

The visualization also has bar charts on hover so that the user can more easily see and
compare how each individual power generation source compares to the total, and the
breakdown. The bar chart here is necessary as the countries are shaded on an absolute scale
relative to the world itself, so the units are not necessarily consistent. The US may produce a lot
of Nuclear Energy, especially in comparison to the rest of the world, but it’s not as much as a
total share of their energy. The bar chart here is more effective than the lower categories, and
one country specificity allows for easier visualization and conception of both how each category
contributes, and the total units along the scale. Whereas the choropleth compares between
countries, a bar chart breaking down energy generation serves as an effective way to see how
each power source compares to itself within a given country.

As for the workflow process, the project began with cloning the Pages Template off of
EdStem, and uploading our dataset of choice, a roughly 30 minute process from Charles. From
there, the next challenge was both finding out and figuring out how to draw a proper world map
from the GeoJSON format, and then figuring out how to link the data from the OWID energy
dataset to their geographic parallels. To facilitate that, Charles used the country’s ISO codes as
an international standard to serve as a link between the two datasets. Overall, this was roughly a
two hour process, with the bulk of the work taken up by figuring out how to work with SVGs and
D3 and the source of the visualization. Afterwards, Jevan added the ability to filter the data by a
specific year, as well as the simple process of shading a specific country that was currently being
hovered, a process that took 3 hours. The bulk of the time was spent on trying to figure out how
to optimize the year selection process, at first a slider was implemented, however later the group
decided best that a text box be implemented instead as there were far to many years to have a
slider. The basic hover function did not take too long to implement and it was a simple as
shading a country green when it was hovered over. Afterwards, he also spent 12 hours expanding
on the interaction on hover, making it so the bar chart breaking down the country’s power
generation sources also appears on mouseover for the data, most of the time spent was having
the bar chart appear next to the cursor, as well as having it be readable. Abishek spent around
20 minutes making the selection of the energy source drop down chart. Finally, Charles spend
around 14 hours figuring out how to shade the data, as the time was spent figuring out how to
filter the data on the provided input, how to calculate the color range based on the provided

column, how to create a proper color scale for D3, and finally how to apply each country the
proper color as provided by said color scale. Finally, roughly an hour was spent figuring out and
categorizing the entire process, culminating in this write up.

Overall, the portion that took the longest was the work involving D3 such as figuring out
how to shade the data and create the bar charts. There were many documentation hours spent
figuring out how D3 creates and applies SVGs, as well as how to update the SVG dynamically
based on user input.
