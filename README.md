<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Krish-e Series Mock Test - Paper II</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            background-color: #f0f4f8;
        }
        .question-grid {
            display: grid;
            grid-template-columns: repeat(4, minmax(0, 1fr));
            grid-template-rows: repeat(25, minmax(0, 2.5rem));
            gap: 0.5rem;
        }
        .question-number {
            width: 2.5rem;
            height: 2.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            font-weight: bold;
        }
        .unattempted {
            background-color: #ef4444;
            color: white;
        }
        .attempted {
            background-color: #22c55e;
            color: white;
        }
        .review {
            background-color: #a855f7;
            color: white;
            position: relative;
        }
        .review::after {
            content: '';
            position: absolute;
            top: -0.25rem;
            right: -0.25rem;
            width: 0.5rem;
            height: 0.5rem;
            background-color: white;
            border-radius: 50%;
        }
        .disabled-btn {
            pointer-events: none;
            opacity: 0.5;
        }
        .meter {
            height: 0.5rem;
            border-radius: 0.25rem;
            background-color: #e5e7eb;
            overflow: hidden;
        }
        .meter-fill {
            height: 100%;
            transition: width 0.3s ease;
        }
        #accuracy-chart {
            max-width: 300px;
            margin: 0 auto;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-green-600 text-white p-4 shadow-md">
        <h1 class="text-3xl font-bold text-center">Krish-e Series</h1>
        <div class="flex justify-between items-center mt-2">
            <div class="flex items-center space-x-4">
                <div>
                    <span class="font-semibold">Time Left: </span>
                    <span id="timer" class="font-mono">02:00:00</span>
                </div>
                <div>
                    <span class="font-semibold">Attempted: </span>
                    <span id="attempted-count">0/100</span>
                </div>
            </div>
        </div>
        <div class="mt-2 flex space-x-4">
            <div class="w-32">
                <span class="text-sm">Attempted Progress</span>
                <div class="meter">
                    <div id="attempted-meter" class="meter-fill bg-green-400"></div>
                </div>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 flex p-6 space-x-6">
        <!-- Question Grid -->
        <aside class="w-1/4 bg-white p-4 rounded-lg shadow-md">
            <h2 class="text-lg font-semibold mb-4">Question Palette</h2>
            <div id="question-grid" class="question-grid"></div>
            <div class="mt-4 text-sm">
                <div class="flex items-center"><span class="w-4 h-4 bg-red-500 mr-2"></span>Unattempted</div>
                <div class="flex items-center"><span class="w-4 h-4 bg-green-500 mr-2"></span>Attempted</div>
                <div class="flex items-center"><span class="w-4 h-4 bg-purple-500 mr-2 relative"><span class="absolute top-0 right-0 w-2 h-2 bg-white rounded-full"></span></span>Save & Review</div>
            </div>
        </aside>

        <!-- Question Display -->
        <section class="flex-1 bg-white p-6 rounded-lg shadow-md">
            <div id="question-container" class="mb-6"></div>
            <div class="flex justify-between">
                <button id="previous-btn" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600 disabled-btn">Previous</button>
                <div class="space-x-2">
                    <button id="save-next-btn" class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">Save and Next</button>
                    <button id="save-review-btn" class="bg-purple-500 text-white px-4 py-2 rounded hover:bg-purple-600">Save and Review</button>
                    <button id="submit-btn" class="bg-red-500 text-white px-4 py-2 rounded hover:bg-red-600">Submit</button>
                </div>
            </div>
        </section>
    </main>

    <!-- Results Section (Hidden Initially) -->
    <section id="results-section" class="hidden p-6 bg-white rounded-lg shadow-md mx-6 mb-6">
        <h2 class="text-2xl font-bold mb-4">Test Results</h2>
        <div class="grid grid-cols-3 gap-4 mb-6">
            <div class="bg-green-100 p-4 rounded">
                <h3 class="font-semibold">Total Score</h3>
                <p id="total-score" class="text-xl"></p>
            </div>
            <div class="bg-blue-100 p-4 rounded">
                <h3 class="font-semibold">Accuracy</h3>
                <p id="final-accuracy" class="text-xl"></p>
            </div>
            <div class="bg-purple-100 p-4 rounded">
                <h3 class="font-semibold">Questions Attempted</h3>
                <p id="final-attempted" class="text-xl"></p>
            </div>
        </div>
        <h3 class="text-xl font-semibold mb-4">Accuracy Breakdown</h3>
        <canvas id="accuracy-chart" class="mb-6"></canvas>
        <h3 class="text-xl font-semibold mb-4">Solutions</h3>
        <div id="solutions-container"></div>
    </section>

    <!-- Footer -->
    <footer class="bg-green-600 text-white text-center p-4">
        <p class="text-sm">By Organic Bharat Education</p>
    </footer>

    <script>
        // Questions Array (All 100 questions from Mock Test 4)
        const questions = [
            {
                id: 1,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> Agro-climatic zones in Karnataka are classified based on rainfall, soil type, and cropping patterns.<br><b>Statement-II:</b> The Coastal Zone of Karnataka is suitable for rice and coconut cultivation due to high rainfall and lateritic soils.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "a"
            },
            {
                id: 2,
                text: `Which of the following factors affect crop weed competition?<br>1. Weed density<br>2. Crop variety<br>3. Soil fertility<br>4. Irrigation method<br>Select the correct answer using the code given below:`,
                options: ["a) 1 and 2 only", "b) 2 and 3 only", "c) 1, 3, and 4 only", "d) 1, 2, 3, and 4"],
                correct: "d"
            },
            {
                id: 3,
                text: `With reference to water use efficiency (WUE) in crop production, consider the following <b>Statements</b>:<br>1. Drip irrigation improves WUE compared to flood irrigation.<br>2. WUE is higher in C4 plants than in C3 plants.<br>Which of the <b>Statements</b> given above is/are correct?`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 4,
                text: `Match the following crops with their primary soil requirement:<br><br>| Crop | Soil Type |<br>| --- | --- |<br>| 1. Rice | A. Sandy loam |<br>| 2. Groundnut | B. Clayey loam |<br>| 3. Sugarcane | C. Well-drained loamy soil |<br>| 4. Cotton | D. Black soil |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "c"
            },
            {
                id: 5,
                text: `Which of the following is a Karnataka-specific initiative launched in 2024 to promote organic farming?`,
                options: ["a) Krishi Bhagya", "b) Raitha Siri", "c) K-KISAN", "d) Bhoomi Suraksha"],
                correct: "d"
            },
            {
                id: 6,
                text: `Consider the following:<br>1. Redgram<br>2. Greengram<br>3. Soybean<br>How many of the above are pulse crops belonging to the Fabaceae family?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 7,
                text: `With reference to precision agriculture, which of the following technologies are used?<br>1. Global Positioning System (GPS)<br>2. Geographic Information System (GIS)<br>3. Remote sensing<br>Select the correct answer using the code given below:`,
                options: ["a) 1 and 2 only", "b) 2 and 3 only", "c) 1 and 3 only", "d) 1, 2, and 3"],
                correct: "d"
            },
            {
                id: 8,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The law of diminishing marginal utility states that as consumption of a good increases, the additional satisfaction derived decreases.<br><b>Statement-II:</b> This principle is applied in agricultural economics to optimize resource allocation.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "b"
            },
            {
                id: 9,
                text: `Which of the following irrigation methods is most suitable for horticultural crops in water-scarce regions of Karnataka?`,
                options: ["a) Flood irrigation", "b) Sprinkler irrigation", "c) Drip irrigation", "d) Canal irrigation"],
                correct: "c"
            },
            {
                id: 10,
                text: `With reference to the 2024 Global Conference on Sustainable Agriculture held in Brazil, which of the following was a key focus area?`,
                options: ["a) Promotion of genetically modified crops", "b) Scaling up natural farming practices", "c) Development of new chemical fertilizers", "d) Expansion of monoculture farming"],
                correct: "b"
            },
            {
                id: 11,
                text: `Consider the following pairs:<br><br>| Insect Pest | Crop Affected |<br>| --- | --- |<br>| 1. Stem borer | Rice |<br>| 2. Pink bollworm | Cotton |<br>| 3. Fruit fly | Mango |<br>| 4. Aphids | Wheat |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "c"
            },
            {
                id: 12,
                text: `Which of the following are components of Integrated Pest Management (IPM)?<br>1. Cultural control<br>2. Chemical control<br>3. Biological control<br>4. Genetic control<br>Select the correct answer using the code given below:`,
                options: ["a) 1 and 2 only", "b) 2 and 3 only", "c) 1, 3, and 4 only", "d) 1, 2, 3, and 4"],
                correct: "d"
            },
            {
                id: 13,
                text: `In the context of agricultural extension, the Training and Visit (T&V) system emphasizes:`,
                options: ["a) Privatization of extension services", "b) Fortnightly meetings and monthly workshops", "c) Digital platforms for farmer training", "d) Community-led crop planning"],
                correct: "b"
            },
            {
                id: 14,
                text: `Consider the following <b>Statements</b> about the Agricultural Technology Management Agency (ATMA) in Karnataka:<br>1. It promotes farmer-scientist interactions through field demonstrations.<br>2. It was restructured in 2023 to include digital extension services.<br>Which of the <b>Statements</b> given above is/are correct?`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 15,
                text: `With reference to agricultural marketing, which of the following is a function of the Agricultural Produce Market Committee (APMC) in India?`,
                options: ["a) Providing crop insurance", "b) Regulating the sale and purchase of agricultural produce", "c) Manufacturing fertilizers", "d) Conducting soil testing"],
                correct: "b"
            },
            {
                id: 16,
                text: `Consider the following:<br>1. National Agricultural Cooperative Marketing Federation of India (NAFED)<br>2. Food Corporation of India (FCI)<br>3. Agricultural and Processed Food Products Export Development Authority (APEDA)<br>How many of the above are involved in agricultural marketing or trade?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 17,
                text: `Which of the following microbes is used as a biofertilizer for nitrogen fixation in leguminous crops?`,
                options: ["a) Rhizobium", "b) Azotobacter", "c) Nitrosomonas", "d) Clostridium"],
                correct: "a"
            },
            {
                id: 18,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> Biological nitrogen fixation is primarily carried out by symbiotic bacteria in root nodules of legumes.<br><b>Statement-II:</b> Non-symbiotic nitrogen fixation is performed by free-living bacteria like Azotobacter.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "b"
            },
            {
                id: 19,
                text: `In the context of agricultural statistics, which of the following tests is used to compare means of two small samples?`,
                options: ["a) Chi-square test", "b) F-test", "c) Student’s t-test", "d) SND test"],
                correct: "c"
            },
            {
                id: 20,
                text: `With reference to the Karnataka State Seed Certification Agency’s 2024 initiative, which of the following aims to enhance seed quality for farmers?`,
                options: ["a) Krishi Samruddhi", "b) Bija Suraksha", "c) Kshetra Vikas", "d) Raitha Bandhu"],
                correct: "b"
            },
            {
                id: 21,
                text: `Consider the following breeds of cattle:<br>1. Gir<br>2. Sahiwal<br>3. Jersey<br>How many of the above are exotic breeds?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "a"
            },
            {
                id: 22,
                text: `Which of the following is a key focus of the 2024 National Livestock Mission in India?`,
                options: ["a) Promoting mechanized farming", "b) Enhancing fodder availability", "c) Developing new pesticides", "d) Expanding sericulture"],
                correct: "b"
            },
            {
                id: 23,
                text: `With reference to apiculture, which of the following <b>Statements</b> is/are correct?<br>1. Apis cerana is a native Indian honeybee species.<br>2. Colony collapse disorder is a major threat to beekeeping globally.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 24,
                text: `Consider the following pairs:<br><br>| Biotechnology Technique | Application |<br>| --- | --- |<br>| 1. Anther culture | Haploid plant production |<br>| 2. Protoplast fusion | Somatic hybridization |<br>| 3. DNA fingerprinting | Pest control |<br>| 4. Marker-assisted selection | Breeding for disease resistance |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "c"
            },
            {
                id: 25,
                text: `Which of the following plant growth regulators is used to induce fruit ripening?`,
                options: ["a) Auxin", "b) Gibberellin", "c) Ethylene", "d) Cytokinin"],
                correct: "c"
            },
            {
                id: 26,
                text: `Consider the following:<br>1. Carbohydrates<br>2. Proteins<br>3. Vitamins<br>4. Minerals<br>How many of the above are macronutrients required in a balanced diet?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "b"
            },
            {
                id: 27,
                text: `With reference to the 2024 International Year of Millets declared by the United Nations, which of the following millets was highlighted for its nutritional benefits?`,
                options: ["a) Foxtail millet", "b) Kodo millet", "c) Barnyard millet", "d) All of the above"],
                correct: "d"
            },
            {
                id: 28,
                text: `Consider the following <b>Statements</b> about the Forest Conservation (Amendment) Act, 2023 in India:<br>1. It allows diversion of forest land for infrastructure projects within 100 km of international borders.<br>2. It exempts certain forest lands from conservation requirements for strategic projects.<br>Which of the <b>Statements</b> given above is/are correct?`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 29,
                text: `Which of the following is a Karnataka-specific afforestation program launched in 2024 to combat desertification?`,
                options: ["a) Harita Karnataka", "b) Vana Samruddhi", "c) Krishi Aranya", "d) Bhoomi Raksha"],
                correct: "b"
            },
            {
                id: 30,
                text: `With reference to Mendelian inheritance, which of the following <b>Statements</b> is/are correct?<br>1. The law of segregation applies to alleles during gamete formation.<br>2. The law of independent assortment applies only to genes on different chromosomes.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 31,
                text: `Consider the following pairs:<br><br>| Crop | Breeding Method |<br>| --- | --- |<br>| 1. Rice | Pedigree method |<br>| 2. Maize | Hybridization |<br>| 3. Cotton | Mass selection |<br>| 4. Wheat | Backcross method |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "d"
            },
            {
                id: 32,
                text: `Which of the following horticultural crops is primarily propagated through stem cuttings?`,
                options: ["a) Mango", "b) Grapes", "c) Banana", "d) Citrus"],
                correct: "b"
            },
            {
                id: 33,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> High-density planting in mango orchards increases yield per unit area.<br><b>Statement-II:</b> It requires intensive pruning and nutrient management.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "a"
            },
            {
                id: 34,
                text: `With reference to plant pathology, which of the following is a symptom of bacterial wilt in tomato?`,
                options: ["a) Yellowing of leaves", "b) Wilting of entire plant", "c) Powdery white coating on leaves", "d) Brown spots on fruits"],
                correct: "b"
            },
            {
                id: 35,
                text: `Which of the following is a Karnataka-specific initiative launched in 2023 to promote integrated disease management in horticultural crops?`,
                options: ["a) Phala Samruddhi", "b) Roga Niyantrana", "c) Krishi Suraksha", "d) Bhoomi Vikas"],
                correct: "b"
            },
            {
                id: 36,
                text: `Consider the following:<br>1. Blast disease in rice<br>2. Rust in wheat<br>3. Downy mildew in grapes<br>How many of the above are caused by fungal pathogens?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 37,
                text: `With reference to seed certification in India, which of the following <b>Statements</b> is/are correct?<br>1. Breeder seed is produced under the supervision of plant breeders.<br>2. Certified seed is produced from foundation seed.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 38,
                text: `Which of the following is a key feature of the UAS Seri-Suvarna Technology developed in Karnataka for rainfed sericulture?`,
                options: ["a) Mechanized cocoon harvesting", "b) Biofertilizer-based mulberry cultivation", "c) Automated silkworm rearing", "d) High-yield hybrid silkworm breeds"],
                correct: "b"
            },
            {
                id: 39,
                text: `Consider the following <b>Statements</b> about soil texture:<br>1. Sandy soils have higher water-holding capacity than clayey soils.<br>2. Loamy soils are ideal for most crops due to balanced texture.<br>Which of the <b>Statements</b> given above is/are correct?`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "b"
            },
            {
                id: 40,
                text: `With reference to the 2024 Global Soil Partnership by FAO, which of the following was emphasized to combat soil degradation?`,
                options: ["a) Use of synthetic fertilizers", "b) Promotion of biochar application", "c) Expansion of monocropping", "d) Development of GM crops"],
                correct: "b"
            },
            {
                id: 41,
                text: `Which of the following is a primary tillage implement?`,
                options: ["a) Plough", "b) Harrow", "c) Cultivator", "d) Seed drill"],
                correct: "a"
            },
            {
                id: 42,
                text: `Consider the following:<br>1. Farmyard manure<br>2. Vermicompost<br>3. Green manure<br>4. Chemical fertilizers<br>How many of the above are organic sources of nutrients?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "c"
            },
            {
                id: 43,
                text: `With reference to agricultural finance, which of the following institutions provides refinancing to cooperative banks?`,
                options: ["a) Reserve Bank of India (RBI)", "b) National Bank for Agriculture and Rural Development (NABARD)", "c) State Bank of India (SBI)", "d) Insurance Regulatory and Development Authority (IRDA)"],
                correct: "b"
            },
            {
                id: 44,
                text: `Which of the following is a key component of watershed management in dryland agriculture?`,
                options: ["a) Use of chemical pesticides", "b) Construction of check dams", "c) Promotion of monoculture", "d) Deep tillage practices"],
                correct: "b"
            },
            {
                id: 45,
                text: `Consider the following pairs:<br><br>| Weed | Crop Affected |<br>| --- | --- |<br>| 1. Parthenium | Cereals |<br>| 2. Cyperus | Rice |<br>| 3. Striga | Sorghum |<br>| 4. Lantana | Horticultural crops |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "d"
            },
            {
                id: 46,
                text: `Which of the following is a Karnataka-specific scheme launched in 2024 to support micro-irrigation in rainfed areas?`,
                options: ["a) Jala Samruddhi", "b) Krishi Neera", "c) Bhoomi Jala", "d) Raitha Jala"],
                correct: "a"
            },
            {
                id: 47,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The Completely Randomized Design (CRD) is suitable for experiments with homogeneous experimental units.<br><b>Statement-II:</b> The Randomized Block Design (RBD) accounts for variability among experimental units.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "b"
            },
            {
                id: 48,
                text: `Which of the following is a characteristic of C4 plants?`,
                options: ["a) High photorespiration", "b) Low water use efficiency", "c) Kranz anatomy", "d) Single carbon fixation pathway"],
                correct: "c"
            },
            {
                id: 49,
                text: `With reference to sericulture, which of the following <b>Statements</b> is/are correct?<br>1. Bombyx mori is the primary mulberry silkworm species.<br>2. Chawki rearing is done for late-age silkworms.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "a"
            },
            {
                id: 50,
                text: `Which of the following is a key focus of the 2024 International Sericulture Congress held in China?`,
                options: ["a) Mechanization of cocoon processing", "b) Development of disease-resistant silkworm breeds", "c) Promotion of synthetic silk", "d) Expansion of non-mulberry sericulture"],
                correct: "b"
            },
            {
                id: 51,
                text: `Consider the following:<br>1. Rhizobium<br>2. Trichoderma<br>3. Bacillus thuringiensis<br>How many of the above are used as biopesticides or biofertilizers?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 52,
                text: `Which of the following is a symptom of nitrogen deficiency in crops?`,
                options: ["a) Stunted growth and yellowing of older leaves", "b) Purple discoloration of leaves", "c) Wilting of young shoots", "d) Excessive vegetative growth"],
                correct: "a"
            },
            {
                id: 53,
                text: `With reference to greenhouse technology, which of the following <b>Statements</b> is/are correct?<br>1. Polyhouses are suitable for year-round cultivation in tropical regions.<br>2. Nutrient Film Technique (NFT) is used for hydroponic farming.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 54,
                text: `Consider the following pairs:<br><br>| Disease | Crop Affected |<br>| --- | --- |<br>| 1. Smut | Sugarcane |<br>| 2. Blight | Potato |<br>| 3. Powdery mildew | Grapes |<br>| 4. Mosaic | Tomato |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "d"
            },
            {
                id: 55,
                text: `Which of the following is a key feature of the National Mission on Sustainable Agriculture (NMSA) in India?`,
                options: ["a) Promotion of chemical-intensive farming", "b) Development of climate-resilient crop varieties", "c) Expansion of large-scale irrigation projects", "d) Subsidies for synthetic pesticides"],
                correct: "b"
            },
            {
                id: 56,
                text: `Consider the following:<br>1. Rose<br>2. Jasmine<br>3. Marigold<br>How many of the above are propagated through cuttings?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "b"
            },
            {
                id: 57,
                text: `Which of the following is a post-harvest preservation method for fruits?`,
                options: ["a) Waxing", "b) Tilling", "c) Pruning", "d) Mulching"],
                correct: "a"
            },
            {
                id: 58,
                text: `With reference to animal science, which of the following breeds is native to Karnataka?`,
                options: ["a) Deoni", "b) Holstein Friesian", "c) Jersey", "d) Red Sindhi"],
                correct: "a"
            },
            {
                id: 59,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The Karnataka Souharda Sahakari Act, 1997 promotes cooperative autonomy.<br><b>Statement-II:</b> It allows cooperatives to function without government interference.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "c"
            },
            {
                id: 60,
                text: `Which of the following is a key component of organic farming?`,
                options: ["a) Use of synthetic fertilizers", "b) Crop rotation", "c) Monocropping", "d) Chemical weed control"],
                correct: "b"
            },
            {
                id: 61,
                text: `With reference to the 2024 National Horticulture Mission, which of the following was a priority area?`,
                options: ["a) Expansion of chemical fertilizer use", "b) Promotion of high-density planting in fruit crops", "c) Development of GM vegetables", "d) Subsidies for monoculture farming"],
                correct: "b"
            },
            {
                id: 62,
                text: `Consider the following:<br>1. Azospirillum<br>2. Pseudomonas<br>3. Mycorrhiza<br>How many of the above are used as biofertilizers?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 63,
                text: `Which of the following is a characteristic of acid soils?`,
                options: ["a) High pH value", "b) Low cation exchange capacity", "c) Aluminum toxicity", "d) High organic matter content"],
                correct: "c"
            },
            {
                id: 64,
                text: `Consider the following <b>Statements</b> about hybrid seed production:<br>1. Male sterility is used in crops like rice and sunflower.<br>2. Foundation seed is used for certified seed production.<br>Which of the <b>Statements</b> given above is/are correct?`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 65,
                text: `Which of the following is a Karnataka-specific watershed management project launched in 2023?`,
                options: ["a) Jala Shakti", "b) Bhoomi Jala", "c) Suvarna Jala", "d) Krishi Jala"],
                correct: "c"
            },
            {
                id: 66,
                text: `With reference to insect morphology, which of the following is a type of insect antennae?`,
                options: ["a) Filiform", "b) Spiracular", "c) Tympanal", "d) Gustatory"],
                correct: "a"
            },
            {
                id: 67,
                text: `Consider the following:<br>1. Contour bunding<br>2. Mulching<br>3. Strip cropping<br>How many of the above are soil conservation techniques?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 68,
                text: `Which of the following is a key feature of the Integrated Farming System (IFS)?`,
                options: ["a) Monoculture farming", "b) Integration of crop and livestock enterprises", "c) Use of chemical pesticides", "d) Large-scale mechanization"],
                correct: "b"
            },
            {
                id: 69,
                text: `With reference to plant tissue culture, which of the following <b>Statements</b> is/are correct?<br>1. Anther culture is used to produce haploid plants.<br>2. Somaclonal variation can introduce genetic diversity.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 70,
                text: `Which of the following is a fungal biofertilizer?`,
                options: ["a) Rhizobium", "b) Azotobacter", "c) Mycorrhiza", "d) Nitrosomonas"],
                correct: "c"
            },
            {
                id: 71,
                text: `Consider the following pairs:<br><br>| Crop | Nutrient Deficiency Symptom |<br>| --- | --- |<br>| 1. Rice | Chlorosis of young leaves |<br>| 2. Maize | Purpling of leaves |<br>| 3. Tomato | Stunted growth and yellowing |<br>| 4. Cotton | Necrosis of leaf margins |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "c"
            },
            {
                id: 72,
                text: `Which of the following is a key focus of the 2024 International Conference on Plant Pathology held in Germany?`,
                options: ["a) Development of new chemical fungicides", "b) Promotion of biocontrol agents", "c) Expansion of GM crops", "d) Use of synthetic fertilizers"],
                correct: "b"
            },
            {
                id: 73,
                text: `With reference to mulberry cultivation for sericulture, which of the following <b>Statements</b> is/are correct?<br>1. Pruning is essential to promote leaf growth.<br>2. Biofertilizers enhance soil fertility for mulberry plants.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 74,
                text: `Consider the following:<br>1. Drip irrigation<br>2. Sprinkler irrigation<br>3. Furrow irrigation<br>How many of the above are suitable for horticultural crops in Karnataka’s dry zones?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "b"
            },
            {
                id: 75,
                text: `Which of the following is a key component of the National Agricultural Technology Project (NATP)?`,
                options: ["a) Promotion of chemical fertilizers", "b) Strengthening research-extension linkages", "c) Expansion of monoculture farming", "d) Subsidies for pesticide use"],
                correct: "b"
            },
            {
                id: 76,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The Seed Act, 1966 regulates the quality of seeds in India.<br><b>Statement-II:</b> It mandates certification for all seeds sold in the market.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "c"
            },
            {
                id: 77,
                text: `Which of the following is a characteristic of Bt crops?`,
                options: ["a) Resistance to fungal diseases", "b) Resistance to insect pests", "c) High water use efficiency", "d) Enhanced seed dormancy"],
                correct: "b"
            },
            {
                id: 78,
                text: `Consider the following:<br>1. Vermicomposting<br>2. Green manuring<br>3. Crop rotation<br>How many of the above are sustainable agriculture practices?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 79,
                text: `Which of the following is a key feature of the Karnataka State Cooperative Societies Act, 1959?`,
                options: ["a) Promotion of private cooperatives", "b) Regulation of cooperative credit structures", "c) Subsidies for chemical fertilizers", "d) Expansion of monoculture farming"],
                correct: "b"
            },
            {
                id: 80,
                text: `With reference to the 2024 Global Biotechnology Summit in the USA, which of the following was a key discussion area?`,
                options: ["a) Development of new synthetic pesticides", "b) Use of CRISPR for crop improvement", "c) Promotion of chemical fertilizers", "d) Expansion of monocropping"],
                correct: "b"
            },
            {
                id: 81,
                text: `Consider the following pairs:<br><br>| Crop | Major Pest |<br>| --- | --- |<br>| 1. Rice | Brown plant hopper |<br>| 2. Cotton | Whitefly |<br>| 3. Mango | Mealybug |<br>| 4. Tomato | Fruit borer |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "d"
            },
            {
                id: 82,
                text: `Which of the following is a method of biological weed control?`,
                options: ["a) Use of herbicides", "b) Introduction of insect predators", "c) Deep tillage", "d) Chemical mulching"],
                correct: "b"
            },
            {
                id: 83,
                text: `With reference to soil microbiology, which of the following <b>Statements</b> is/are correct?<br>1. Rhizobium fixes nitrogen in symbiosis with legumes.<br>2. Azotobacter is a free-living nitrogen-fixing bacterium.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 84,
                text: `Consider the following:<br>1. Tractor<br>2. Seed drill<br>3. Sprayer<br>How many of the above are mechanized farm implements?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 85,
                text: `Which of the following is a key feature of the Raitha Samparka Kendras (RSKs) in Karnataka?`,
                options: ["a) Distribution of chemical fertilizers", "b) Farmer training and extension services", "c) Seed certification", "d) Crop insurance"],
                correct: "b"
            },
            {
                id: 86,
                text: `With reference to post-harvest technology, which of the following is used to extend the shelf life of mangoes?`,
                options: ["a) Waxing", "b) Tilling", "c) Pruning", "d) Mulching"],
                correct: "a"
            },
            {
                id: 87,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The chi-square test is used to analyze categorical data in agricultural experiments.<br><b>Statement-II:</b> It is suitable for testing the goodness of fit of observed data to expected ratios.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "a"
            },
            {
                id: 88,
                text: `Which of the following is a characteristic of organic farming?`,
                options: ["a) Use of synthetic pesticides", "b) Crop monoculture", "c) Biological pest control", "d) Chemical fertilizer application"],
                correct: "c"
            },
            {
                id: 89,
                text: `With reference to livestock management, which of the following <b>Statements</b> is/are correct?<br>1. Crossbreeding improves milk yield in cattle.<br>2. Vaccination is essential for disease prevention in poultry.<br>Select the correct answer using the code given below:`,
                options: ["a) 1 only", "b) 2 only", "c) Both 1 and 2", "d) Neither 1 nor 2"],
                correct: "c"
            },
            {
                id: 90,
                text: `Which of the following is a key focus of the 2024 National Mission on Oilseeds and Oil Palm?`,
                options: ["a) Promotion of chemical fertilizers", "b) Expansion of oilseed cultivation in rainfed areas", "c) Development of GM oilseeds", "d) Subsidies for monoculture farming"],
                correct: "b"
            },
            {
                id: 91,
                text: `Consider the following:<br>1. Arecanut<br>2. Coconut<br>3. Pepper<br>How many of the above are plantation crops grown in Karnataka?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 92,
                text: `Which of the following is a method of breaking seed dormancy?`,
                options: ["a) Scarification", "b) Mulching", "c) Tilling", "d) Pruning"],
                correct: "a"
            },
            {
                id: 93,
                text: `With reference to agricultural extension, which of the following is a group contact method?`,
                options: ["a) Farm visit", "b) Method demonstration", "c) Radio broadcast", "d) Kisan call center"],
                correct: "b"
            },
            {
                id: 94,
                text: `Consider the following <b>Statements</b>:<br><b>Statement-I:</b> The Forest Conservation Act, 1980 regulates the diversion of forest land for non-forest purposes.<br><b>Statement-II:</b> It requires compensatory afforestation for approved projects.<br>Which one of the following is correct in respect of the above <b>Statements</b>?`,
                options: ["a) Both Statement-I and Statement-II are correct, and Statement-II explains Statement-I", "b) Both Statement-I and Statement-II are correct, but Statement-II does not explain Statement-I", "c) Statement-I is correct, but Statement-II is incorrect", "d) Statement-I is incorrect, but Statement-II is correct"],
                correct: "b"
            },
            {
                id: 95,
                text: `Which of the following is a Karnataka-specific initiative launched in 2024 to promote sericulture?`,
                options: ["a) Reshme Krishi", "b) Seri Samruddhi", "c) Kshetra Siri", "d) Bhoomi Reshme"],
                correct: "b"
            },
            {
                id: 96,
                text: `Consider the following:<br>1. Transpiration<br>2. Photosynthesis<br>3. Respiration<br>How many of the above physiological processes are affected by water stress in plants?`,
                options: ["a) Only one", "b) Only two", "c) All three", "d) None"],
                correct: "c"
            },
            {
                id: 97,
                text: `Which of the following is a characteristic of saline soils?`,
                options: ["a) High organic matter content", "b) Low electrical conductivity", "c) High sodium content", "d) Low pH value"],
                correct: "c"
            },
            {
                id: 98,
                text: `With reference to apiculture, which of the following is a common bee disease?`,
                options: ["a) Nosema", "b) Rust", "c) Blight", "d) Wilt"],
                correct: "a"
            },
            {
                id: 99,
                text: `Consider the following pairs:<br><br>| Crop | Harvesting Method |<br>| --- | --- |<br>| 1. Rice | Combine harvester |<br>| 2. Mango | Hand picking |<br>| 3. Cotton | Mechanical picker |<br>| 4. Sugarcane | Manual cutting |<br><br>How many of the above pairs are correctly matched?`,
                options: ["a) Only one", "b) Only two", "c) Only three", "d) All four"],
                correct: "d"
            },
            {
                id: 100,
                text: `Which of the following is a key feature of the National Agricultural Innovation Project (NAIP)?`,
                options: ["a) Promotion of chemical-intensive farming", "b) Strengthening agricultural research and innovation", "c) Expansion of monoculture farming", "d) Subsidies for synthetic pesticides"],
                correct: "b"
            }
        ];

        // Randomize Questions
        const randomizedQuestions = questions.sort(() => Math.random() - 0.5).map((q, index) => ({ ...q, displayId: index + 1 }));

        // State Management
        let currentQuestionIndex = 0;
        let answers = Array(questions.length).fill(null); // Stores selected answers
        let statuses = Array(questions.length).fill('unattempted'); // Tracks question status
        let timeLeft = 2 * 60 * 60; // 2 hours in seconds
        let timerInterval;

        // DOM Elements
        const timerEl = document.getElementById('timer');
        const attemptedCountEl = document.getElementById('attempted-count');
        const attemptedMeterEl = document.getElementById('attempted-meter');
        const questionGridEl = document.getElementById('question-grid');
        const questionContainerEl = document.getElementById('question-container');
        const previousBtn = document.getElementById('previous-btn');
        const saveNextBtn = document.getElementById('save-next-btn');
        const saveReviewBtn = document.getElementById('save-review-btn');
        const submitBtn = document.getElementById('submit-btn');
        const resultsSection = document.getElementById('results-section');
        const totalScoreEl = document.getElementById('total-score');
        const finalAccuracyEl = document.getElementById('final-accuracy');
        const finalAttemptedEl = document.getElementById('final-attempted');
        const solutionsContainerEl = document.getElementById('solutions-container');
        const accuracyChartEl = document.getElementById('accuracy-chart');

        // Initialize Test
        function init() {
            console.log('Initializing test...');
            try {
                if (!questionContainerEl) {
                    console.error('Question container not found!');
                    return;
                }
                console.log('Rendering question grid...');
                renderQuestionGrid();
                console.log('Rendering first question...');
                renderQuestion();
                console.log('Starting timer...');
                startTimer();
                console.log('Updating meters...');
                updateMeters();
                console.log('Adding beforeunload listener...');
                window.addEventListener('beforeunload', handleBeforeUnload);
                console.log('Initialization complete.');
            } catch (error) {
                console.error('Initialization error:', error);
            }
        }

        // Timer
        function startTimer() {
            console.log('Starting timer...');
            timerInterval = setInterval(() => {
                timeLeft--;
                if (timeLeft <= 0) {
                    clearInterval(timerInterval);
                    submitTest();
                }
                const hours = Math.floor(timeLeft / 3600);
                const minutes = Math.floor((timeLeft % 3600) / 60);
                const seconds = timeLeft % 60;
                timerEl.textContent = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
            }, 1000);
        }

        // Render Question Grid
        function renderQuestionGrid() {
            console.log('Rendering question grid...');
            try {
                if (!questionGridEl) {
                    console.error('Question grid element not found!');
                    return;
                }
                questionGridEl.innerHTML = '';
                randomizedQuestions.forEach((_, index) => {
                    const div = document.createElement('div');
                    div.className = `question-number ${statuses[index]} ${index === currentQuestionIndex ? 'ring-2 ring-blue-500' : ''}`;
                    div.textContent = index + 1;
                    div.addEventListener('click', () => {
                        console.log(`Navigating to question ${index + 1}`);
                        currentQuestionIndex = index;
                        renderQuestion();
                        renderQuestionGrid();
                    });
                    questionGridEl.appendChild(div);
                });
            } catch (error) {
                console.error('Error rendering question grid:', error);
            }
        }

        // Render Current Question
        function renderQuestion() {
            console.log(`Rendering question ${currentQuestionIndex + 1}`);
            try {
                const question = randomizedQuestions[currentQuestionIndex];
                if (!question) {
                    console.error(`Question at index ${currentQuestionIndex} not found!`);
                    questionContainerEl.innerHTML = '<p>Error: Question not found.</p>';
                    return;
                }
                questionContainerEl.innerHTML = `
                    <h3 class="text-lg font-semibold mb-4">Question ${currentQuestionIndex + 1}</h3>
                    <p class="mb-4">${question.text}</p>
                    <div class="space-y-2">
                        ${question.options.map((option, idx) => `
                            <label class="flex items-center space-x-2">
                                <input type="radio" name="answer" value="${option.charAt(0)}" ${answers[currentQuestionIndex] === option.charAt(0) ? 'checked' : ''}>
                                <span>${option}</span>
                            </label>
                        `).join('')}
                    </div>
                `;
                previousBtn.classList.toggle('disabled-btn', currentQuestionIndex === 0);
                saveNextBtn.textContent = currentQuestionIndex === randomizedQuestions.length - 1 ? 'Save' : 'Save and Next';
            } catch (error) {
                console.error('Error rendering question:', error);
                questionContainerEl.innerHTML = '<p>Error: Failed to render question.</p>';
            }
        }

        // Update Meters
        function updateMeters() {
            console.log('Updating meters...');
            try {
                const attempted = answers.filter(a => a !== null).length;
                attemptedCountEl.textContent = `${attempted}/${randomizedQuestions.length}`;
                attemptedMeterEl.style.width = `${(attempted / randomizedQuestions.length) * 100}%`;
            } catch (error) {
                console.error('Error updating meters:', error);
            }
        }

        // Save and Next
        saveNextBtn.addEventListener('click', () => {
            console.log('Save and Next clicked');
            try {
                const selected = document.querySelector('input[name="answer"]:checked');
                if (selected) {
                    answers[currentQuestionIndex] = selected.value;
                    statuses[currentQuestionIndex] = 'attempted';
                }
                if (currentQuestionIndex < randomizedQuestions.length - 1) {
                    currentQuestionIndex++;
                }
                renderQuestion();
                renderQuestionGrid();
                updateMeters();
            } catch (error) {
                console.error('Error in Save and Next:', error);
            }
        });

        // Previous
        previousBtn.addEventListener('click', () => {
            console.log('Previous clicked');
            try {
                if (currentQuestionIndex > 0) {
                    currentQuestionIndex--;
                    renderQuestion();
                    renderQuestionGrid();
                }
            } catch (error) {
                console.error('Error in Previous:', error);
            }
        });

        // Save and Review
        saveReviewBtn.addEventListener('click', () => {
            console.log('Save and Review clicked');
            try {
                const selected = document.querySelector('input[name="answer"]:checked');
                if (selected) {
                    answers[currentQuestionIndex] = selected.value;
                }
                statuses[currentQuestionIndex] = 'review';
                if (currentQuestionIndex < randomizedQuestions.length - 1) {
                    currentQuestionIndex++;
                }
                renderQuestion();
                renderQuestionGrid();
                updateMeters();
            } catch (error) {
                console.error('Error in Save and Review:', error);
            }
        });

        // Submit Test
        submitBtn.addEventListener('click', () => {
            console.log('Submit button clicked');
            try {
                submitTest();
            } catch (error) {
                console.error('Error in Submit:', error);
            }
        });

        function submitTest() {
            console.log('Submitting test...');
            try {
                clearInterval(timerInterval);
                window.removeEventListener('beforeunload', handleBeforeUnload);
                let score = 0;
                let correct = 0;
                let attempted = 0;
                answers.forEach((ans, idx) => {
                    if (ans) {
                        attempted++;
                        if (ans === randomizedQuestions[idx].correct) {
                            score += 3;
                            correct++;
                        } else {
                            score -= 1;
                        }
                    }
                });
                const accuracy = attempted > 0 ? (correct / attempted * 100).toFixed(2) : 0;
                totalScoreEl.textContent = `${score.toFixed(2)} / ${randomizedQuestions.length * 3}`;
                finalAccuracyEl.textContent = `${accuracy}%`;
                finalAttemptedEl.textContent = `${attempted} / ${randomizedQuestions.length}`;
                solutionsContainerEl.innerHTML = randomizedQuestions.map((q, idx) => `
                    <div class="mb-6 p-4 bg-gray-50 rounded">
                        <h4 class="font-semibold">Question ${idx + 1}</h4>
                        <p class="mb-2">${q.text}</p>
                        <p><b>Your Answer:</b> ${answers[idx] || 'Not Attempted'}</p>
                        <p><b>Correct Answer:</b> ${q.correct}</p>
                        <p><b>Options:</b></p>
                        <ul class="list-disc pl-5">
                            ${q.options.map(opt => `<li>${opt}</li>`).join('')}
                        </ul>
                    </div>
                `).join('');

                // Render Pie Chart
                console.log('Rendering pie chart...');
                const incorrect = attempted - correct;
                new Chart(accuracyChartEl, {
                    type: 'pie',
                    data: {
                        labels: ['Correct', 'Incorrect'],
                        datasets: [{
                            data: [correct, incorrect],
                            backgroundColor: ['#22c55e', '#ef4444'],
                            borderColor: ['#ffffff', '#ffffff'],
                            borderWidth: 1
                        }]
                    },
                    options: {
                        responsive: true,
                        plugins: {
                            legend: {
                                position: 'top'
                            },
                            title: {
                                display: true,
                                text: 'Accuracy Breakdown'
                            }
                        }
                    }
                });

                resultsSection.classList.remove('hidden');
                document.querySelector('main').classList.add('hidden');
                window.scrollTo(0, 0);
            } catch (error) {
                console.error('Error submitting test:', error);
            }
        }

        // Prevent Tab Exit
        function handleBeforeUnload(e) {
            console.log('Attempted to leave page');
            e.preventDefault();
            e.returnValue = 'You can only exit after submitting!';
        }

        // Start the Test
        document.addEventListener('DOMContentLoaded', () => {
            console.log('DOM fully loaded, starting test...');
            init();
        });
    </script>
</body>
</html>
