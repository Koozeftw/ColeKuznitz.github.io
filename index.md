
# Investigating the Relationship Between Cuisine and Recipe Rating
Author: Cole Kuznitz  

## Introduction {#introduction}
Everyone on this planet must eat, but what each person eats varies.  For many people, they will look online to find meals that entice them, often those that are rated the highest. When searching recipes in google, Food.com is the fifth link that comes up and upon going on Food.com and clicking on a recipe, the rating is at the top of the screen, making it easy to screen the good recipes from the bad recipes.  My goal was to find if the recipe's rating was dependent on the cuisine of the dish.  To do this I am going to use two datasets, the Recipes and Interactions dataset which have both been scraped from Food.com. The Recipes dataset contains a column called 'id' which contains the id of the recipe, and the Interactions data contains a similar column called 'recipe_id.'  Upon renaming 'id' to 'recipe_id' it was possible to left merge the Interactions dataset onto the Recipe dataset, resulting in a merged dataset with 234429 rows amd 17 columns.  The most relevent columns in this dataset are: 'name', a column containing the name of the dish as a string, 'rating', a column containing a float value from 1 to 5 describing the rating in stars that a recipe received, 'nutrition', a column containing a string representation of the float values off calories in a dish and then total_fat, saturated_fat, protein, carbohydrates, sugar, and sodium in units of percent daily values (PDV), and 'contributor_id', a column containing an integer of the id of the person that added the recipe to Food.com.  

## Data Cleaning and Exploratory Analysis {#datacleaningandexploratoryanalysis}
Data cleaning began with replacing all of the 0 ratings in the 'rating' column with a Nan value.  This is due to a 0 rating being outside of the rating scale of the site, so a 0 score is a missing rating.  By changing these values to Nan values it allows for the exclusion of these recipes which have not received a valid rating in aggregation, as well as, for consideration in further hypothesis testing. Duplicates were then dropped, which ensures that each data point is unique and that certain categories can't be unfairly weighted due to having artificial weighting from the same thing being represented multiple times.  The column average rating was then added, which is the float value of the rating for each 'recipe_id'.  This value gives a concise metric for each recipe and is the float representation of what would be seen by individuals as they look at the recipe on Food.com.  The nutrition column was then split into different columns based on the data contained within, with a calories, total_fat, sodium, sugar, saturated_fat, protein, and carbohydrates.  Each of these new columns contains float values, and each column besides calories was updated to be in units of grams instead of PDV.  This was done based on a recommended daily intake of 70g of total fat, 20g of saturated fat, 2.3g of sodium, 70g of protein, 130g of carbohydrates, and 50g of sugar.  This allows for easier access to each individual macromolecule, as well as, more easily understood values as the average person will resonate more with a recipe that has 50g of protein, instead of saying that a recipe has 71% of their daily value of protein.  PDV is also highly individualistic while grams is a universal unit of measurement.  The next step was to create a column named 'tokenized' which contained a list of strings where each name was broken into tokens.  Using this column it was then possible to sort each dish into a cuisine using this dictionary:  
<details>
<summary><strong>Click to expand cuisine dictionary</strong></summary>

```python
cuisine_keywords = {
    'italian': ['pasta', 'risotto', 'lasagna', 'gnocchi', 'carbonara', 'meatball', 'ziti', 'pizza', 'chicken parmigiana', 'chicken parm', 'eggplant parm', 'italian', 'macaroni', 'bolognese', 'caprese', 'marinara', 'alfredo', 'penne', 'fettuccine', 'spaghetti', 'prosciutto', 'bruschetta', 'calzone', 'tortellini'],
    'mexican': ['taco', 'burrito', 'quesadilla', 'enchilada', 'salsa', 'carne asada', 'tacos', 'burritos', 'mexican', 'fajita', 'guacamole', 'tamale', 'chilaquiles', 'pozole', 'elote', 'horchata', 'menudo', 'mole', 'sope', 'taquito'],
    'indian': ['masala', 'curry', 'tikka', 'naan', 'dal', 'samosas', 'indian', 'biryani', 'paneer', 'chana', 'rogan', 'roti', 'kachori', 'bhaji', 'pulao', 'idli', 'dosas', 'vindaloo', 'chaat'],
    'japanese': ['sushi', 'teriyaki', 'ramen', 'udon', 'tonkatsu', 'karaage', 'japanese', 'sashimi', 'miso', 'yakitori', 'tempura', 'bento', 'onigiri', 'gyoza', 'matcha'],
    'chinese': ['dumpling', 'kung', 'noodle', 'lo', 'mein', 'peking duck', 'pork intestine', 'dim sum', 'firecracker', 'chinese', 'wonton', 'mapo', 'chow mein', 'hot pot', 'scallion pancake', 'szechuan', 'general tso', 'char siu'],
    'french': ['crepe', 'ratatouille', 'baguette', 'souffle', 'escargot', 'french', 'coq au vin', 'bouillabaisse', 'cassoulet', 'croissant', 'quiche', 'tarte', 'beurre blanc'],
    'dessert': ['cake','brownies', 'cookies', 'candy', 'ice cream', 'sundae', 'cookie', 'brownie', 'candies', 'mango sticky rice', 'dessert'],
    'american': ['casserole', 'burger', 'hamburger', 'hamburgers', 'burgers', 'cheeseburger', 'cheeseburgers', 'fries', 'fried', 'grits', 'mac and cheese', 'lobster roll', 'maine lobster', 'seafood boil', 'barbecue', 'bbq', 'ribs', 'smoked', 'chili', 'american', 'cornbread', 'sloppy joe', 'biscuits', 'meatloaf', 'tater tots', 'jambalaya', 'corn dog'],
    'english': ['roast', 'toast', 'mashed potatoes', 'shepherd pie', 'english', 'tart', 'swiss', 'yorkshire', 'pudding', 'scone', 'mince pie', 'bangers', 'fish and chips', 'trifle'],
    'thai': ['tam', 'pad thai', 'tom yum soup', 'thai', 'larb', 'green curry', 'red curry', 'massaman', 'satay'],
    'vietnamese': ['pho', 'bahn', 'fishcake', 'bun cha', 'banh mi', 'goi cuon', 'ca kho', 'cha gio'],
    'mediterranean': ['greek', 'hummus', 'kebab', 'egyptian', 'shakshuka', 'briam', 'tabouli', 'grilled swordfish', 'mediterranean', 'tzatziki', 'falafel', 'dolma', 'spanakopita', 'shawarma', 'labneh'],
    'breakfast': ['egg', 'waffle', 'pancake', 'hashbrowns', 'breakfast', 'bacon', 'scrambled', 'frittata', 'oatmeal', 'omelet', 'granola'],
    'african': ['zydeco', 'jollof', 'ugali', 'bobotie', 'peri peri', 'yassa', 'african', 'fufu', 'couscous', 'ethiopian', 'ghanian', 'injera', 'tagine', 'bunny chow', 'suya', 'berbere', 'koshari'],
    'german': ['zw', 'bratwurst', 'schnitzel', 'wurst', 'goulash', 'zu', 'spaetzle', 'strudel', 'currywurst', 'sauerbraten', 'pretzel'],
    'korean': ['kimchi', 'bibimbap', 'bulgogi', 'jjigae', 'kimbap', 'gochujang', 'galbi', 'tteokbokki'],
    'spanish': ['paella', 'tapas', 'gazpacho', 'churros', 'tortilla española', 'patatas bravas'],
    'brazilian': ['feijoada', 'pão de queijo', 'brigadeiro', 'moqueca', 'coxinha'],
    'turkish': ['baklava', 'doner', 'lahmacun', 'pide', 'borek', 'kofte'],
    'indonesian': ['rendang', 'nasi goreng', 'sate', 'gado-gado'],
    'middle eastern': ['mansaf', 'kofta', 'fattoush', 'mutabbal', 'kibbeh', 'maqluba']
}
```
</details>


## Assessment of Missingness {#assessmentofmissingness}
blah blah blah 

## Hypothesis Testing {#hypothesistesting}
blah blah blah

## Framing a Prediction Problem {#framingapredictionproblem}
blah blah blah 

## Baseline Model {#baselinemodel}
blah blah blah

## Final Model {#finalmodel}
blah blah blah

## Fairness Analysis {#fairnessanalysis}
