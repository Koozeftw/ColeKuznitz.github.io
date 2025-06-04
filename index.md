<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Investigating the Relationship Between Cuisine and Recipe Rating</title>
    <style>
        body { font-family: Arial, sans-serif; line-height: 1.6; margin: 40px; }
        h2 { margin-top: 2em; }
        table { border-collapse: collapse; width: 100%; margin-top: 1em; }
        table, th, td { border: 1px solid #ccc; }
        th, td { padding: 8px; text-align: left; }
        summary { font-weight: bold; cursor: pointer; }
        pre { background-color: #f4f4f4; padding: 10px; overflow: auto; }
    </style>
</head>
<body>

<h1>Investigating the Relationship Between Cuisine and Recipe Rating</h1>
<p><strong>Author:</strong> Cole Kuznitz</p>

<h2 id="introduction">Introduction</h2>
<p>Everyone on this planet must eat, but what each person eats varies. For many people, they will look online to find meals that entice them, often those that are rated the highest. When searching recipes in google, Food.com is the fifth link that comes up and upon going on Food.com and clicking on a recipe, the rating is at the top of the screen, making it easy to screen the good recipes from the bad recipes.</p>

<p>My goal was to find if the recipe's rating was dependent on the cuisine of the dish. To do this I am going to use two datasets, the Recipes and Interactions dataset which have both been scraped from Food.com. The Recipes dataset contains a column called 'id' which contains the id of the recipe, and the Interactions data contains a similar column called 'recipe_id.' Upon renaming 'id' to 'recipe_id' it was possible to left merge the Interactions dataset onto the Recipe dataset, resulting in a merged dataset with 234429 rows and 17 columns.</p>

<p>The most relevant columns in this dataset are: 'name', a column containing the name of the dish as a string, 'rating', a column containing a float value from 1 to 5 describing the rating in stars that a recipe received, 'nutrition', a column containing a string representation of the float values of calories in a dish and then total_fat, saturated_fat, protein, carbohydrates, sugar, and sodium in units of percent daily values (PDV), and 'contributor_id', a column containing an integer of the id of the person that added the recipe to Food.com.</p>

<h2 id="datacleaningandexploratoryanalysis">Data Cleaning and Exploratory Analysis</h2>
<p>Data cleaning began with replacing all of the 0 ratings in the 'rating' column with a NaN value. This is due to a 0 rating being outside of the rating scale of the site, so a 0 score is a missing rating. By changing these values to NaN values it allows for the exclusion of these recipes which have not received a valid rating in aggregation, as well as for consideration in further hypothesis testing.</p>

<p>Duplicates were then dropped, which ensures that each data point is unique and that certain categories can't be unfairly weighted due to having artificial weighting from the same thing being represented multiple times.</p>

<p>The column average rating was then added, which is the float value of the rating for each 'recipe_id'. The nutrition column was then split into different columns based on the data contained within, including calories, total_fat, sodium, sugar, saturated_fat, protein, and carbohydrates. Each of these new columns contains float values, and each column besides calories was updated to be in units of grams instead of PDV.</p>

<p>The next step was to create a column named 'tokenized' which contained a list of strings where each name was broken into tokens. Using this column it was then possible to sort each dish into a cuisine using a predefined dictionary.</p>

<details>
<summary><strong>Click to expand cuisine dictionary</strong></summary>
<pre>
cuisine_keywords = {
    'italian': ['pasta', 'risotto', 'lasagna', 'gnocchi', 'carbonara', 'meatball', 'ziti', 'pizza', 'chicken parmigiana', 'chicken parm', 'eggplant parm', 'italian', 'macaroni', 'bolognese', 'caprese', 'marinara', 'alfredo', 'penne', 'fettuccine', 'spaghetti', 'prosciutto', 'bruschetta', 'calzone', 'tortellini'],
    'mexican': ['taco', 'burrito', 'quesadilla', 'enchilada', 'salsa', 'carne asada', 'tacos', 'burritos', 'mexican', 'fajita', 'guacamole', 'tamale', 'chilaquiles', 'pozole', 'elote', 'horchata', 'menudo', 'mole', 'sope', 'taquito'],
    'indian': ['masala', 'curry', 'tikka', 'naan', 'dal', 'naan', 'samosas', 'indian', 'biryani', 'paneer', 'chana', 'rogan', 'roti', 'kachori', 'bhaji', 'pulao', 'idli', 'dosas', 'vindaloo', 'chaat'],
    'japanese': ['sushi', 'teriyaki', 'ramen', 'udon', 'tonkatsu', 'karaage', 'japanese', 'sashimi', 'miso', 'yakitori', 'tempura', 'bento', 'onigiri', 'gyoza', 'matcha'],
    'chinese': ['dumpling', 'kung', 'noodle', 'lo', 'mein', 'peking duck', 'pork intestine', 'dim sum', 'firecracker', 'chinese', 'wonton', 'mapo', 'chow mein', 'hot pot', 'scallion pancake', 'szechuan', 'general tso', 'char siu'],
    'french': ['crepe', 'ratatouille', 'baguette', 'souffle', 'escargot', 'french', 'coq au vin', 'bouillabaisse', 'cassoulet', 'croissant', 'quiche', 'tarte', 'beurre blanc'],
    'dessert': ['cake','brownies', 'cookies', 'candy', 'ice cream', 'sundae', 'cookie', 'brownie', 'candies', 'mango sticky rice', 'dessert'],
    'american': ['casserole', 'burger', 'hamburger', 'hamburgers', 'burgers', 'cheeseburger', 'cheeseburgers', 'fries', 'fried', 'grits', 'mac and cheese', 'lobster roll', 'maine lobster', 'seafood boil', 'barbecue', 'bbq', 'ribs', 'smoked', 'chili', 'american', 'cornbread', 'sloppy joe', 'biscuits', 'meatloaf', 'tater tots', 'jambalaya', 'corn dog'],
    'english': ['roast', 'toast', 'mashed potatoes', 'shepherd pie', 'english', 'tart', 'swiss', 'yorkshire', 'pudding', 'scone', 'mince pie', 'bangers', 'fish and chips', 'trifle'],
    'thai': ['tam', 'pad thai', 'tom yum soup', 'thai', 'larb', 'green curry', 'red curry', 'massaman', 'satay'],
    'vietnamese': ['pho', 'bahn', 'fishcake', 'bun cha', 'banh mi', 'goi cuon', 'ca kho', 'cha gio'],
    'mediterranean': ['greek', 'hummus', 'kebab', 'hummus', 'egyptian', 'shakshuka', 'briam', 'tabouli', 'grilled swordfish', 'mediterranean', 'tzatziki', 'falafel', 'dolma', 'spanakopita', 'shawarma', 'labneh'],
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
</pre>
</details>

<p>This dictionary allowed for the creation of the 'cuisine' column, which contained the sorted cuisine for each dish stored as a string, and 'other' for any dish that didn't fall into any of those categories.</p>

<p>Unnecessary columns were dropped from the dataframe, including 'tokenized', 'nutrition', 'index', 'review', 'submitted', 'tags', 'n_steps', 'description', 'user_id', 'n_ingredients', 'minutes', 'steps', and 'date'. This left a total of 13 columns in the dataframe, with our final cleaned, merged dataframe looking like this:</p>

<div style="overflow-x: auto; max-height: 400px; border: 1px solid #ccc; margin-top: 1em;">
    <table>
        <thead>
            <tr>
                <th>name</th>
                <th>recipe_id</th>
                <th>contributor_id</th>
                <th>rating</th>
                <th>average_rating</th>
                <th>calories</th>
                <th>total_fat</th>
                <th>sugar</th>
                <th>sodium</th>
                <th>protein</th>
                <th>saturated_fat</th>
                <th>carbohydrates</th>
                <th>cuisine</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1 brownies in the world best ever</td>
                <td>333281</td>
                <td>985201</td>
                <td>4</td>
                <td>4</td>
                <td>138.4</td>
                <td>7</td>
                <td>25</td>
                 <td>0.069</td>
                <td>2.1</td>
                <td>3.8</td>
                <td>7.8</td>
                <td>dessert</td>
            </tr>
            <tr>
                <td>1 in canada chocolate chip cookies</td>
                <td>453467</td>
                <td>1848091</td>
                <td>5</td>
                <td>5</td>
                <td>595.1</td>
                <td>32.2</td>
                <td>105.5</td>
                <td>0.506</td>
                <td>9.1</td>
                <td>10.2</td>
                <td>33.8</td>
                <td>dessert</td>
            </tr>
            <tr>
                <td>412 broccoli casserole</td>
                <td>306168</td>
                <td>50969</td>
                <td>5</td>
                <td>5</td>
                <td>194.8</td>
                <td>14</td>
                 <td>3</td>
                <td>0.736</td>
                <td>15.4</td>
                <td>7.2</td>
                <td>3.9</td>
                <td>american</td>
            </tr>
            <tr>
                <td>millionaire pound cake</td>
                <td>286009</td>
                <td>461724</td>
                <td>5</td>
                <td>5</td>
                <td>878.3</td>
                <td>44.1</td>
                <td>163</td>
                <td>0.299</td>
                <td>14</td>
                <td>24.6</td>
                <td>50.7</td>
                <td>dessert</td>
            </tr>
            <tr>
                <td>2000 meatloaf</td>
                <td>475785</td>
                <td>2202916</td>
                <td>5</td>
                <td>5</td>
                <td>267</td>
                <td>21</td>
                <td>6</td>
                <td>0.276</td>
                <td>20.3</td>
                <td>9.6</td>
                <td>2.6</td>
                <td>chinese</td>
            </tr>
        </tbody>
    </table>
</div>
<h2 id="assessmentofmissingness">Assessment of Missingness</h2>
<p>blah blah blah</p>

<h2 id="hypothesistesting">Hypothesis Testing</h2>
<p>blah blah blah</p>

<h2 id="framingapredictionproblem">Framing a Prediction Problem</h2>
<p>blah blah blah</p>

<h2 id="baselinemodel">Baseline Model</h2>
<p>blah blah blah</p>

<h2 id="finalmodel">Final Model</h2>
<p>blah blah blah</p>

<h2 id="fairnessanalysis">Fairness Analysis</h2>
<p>blah blah blah</p>

</body>
</html>
