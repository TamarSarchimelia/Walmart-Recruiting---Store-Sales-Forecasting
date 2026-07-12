# Walmart-Recruiting---Store-Sales-Forecasting

1. პროექტის მიზანი
ამ პროექტის მიზანია Walmart-ის მაღაზიების ყოველკვირეული გაყიდვების პროგნოზირება სხვადასხვა time-series მოდელების გამოყენებით და მათი შედარებითი ანალიზი.
მონაცემები წარმოადგენს მრავალ მაღაზიასა და დეპარტამენტზე განაწილებულ დროით სერიებს, სადაც მნიშვნელოვანი ფაქტორებია სეზონურობა, ტრენდი და გარე მოვლენებით გამოწვეული ცვლილებები (მაგალითად: დღესასწაულები და ფასდაკლებები).
პროექტის ძირითადი ამოცანაა სხვადასხვა არქიტექტურის შეფასება და იმის განსაზღვრა, თუ რომელი მიდგომა უკეთ ერგება ასეთი ტიპის რთულ, მრავალცვლადიან time-series პრობლემას.

2. მონაცემების აღწერა და წინასწარი დამუშავება
მონაცემები აღებულია Kaggle კონკურსიდან: Walmart Recruiting - Store Sales Forecasting.
მონაცემთა სტრუქტურა მოიცავს:
  Store და Dept იდენტიფიკატორებს 
  Weekly_Sales (სამიზნე ცვლადი) 
  თარიღს (weekly frequency) 
  დამატებით გარე ფაქტორებს: Temperature, Fuel Price, CPI, Unemployment და Holiday indicators 

მონაცემები გადაყვანილია unified time-series ფორმატში:
  unique_id = Store_Dept 
  ds = Date 
  y = Weekly_Sales 

შემდგომ განხორციელდა:
  მონაცემების გაერთიანება (train, features, stores) 
  დროითი დალაგება 
  missing values-ის დამუშავება 
  train/validation split დროითი ლოგიკით 

3. გამოყენებული მიდგომები
პროექტში გამოყენებულია განსხვავებული მეთოდოლოგია:
  ARIMA (კლასიკური სტატისტიკური მოდელი)
  SARIMA (სეზონურობაზე გაფართოებული ARIMA)
  DLinear (მსუბუქი deep learning არქიტექტურა)
  N-BEATS (ღრმა ნეირონული forecasting მოდელი)
  LightGBM (LGBM) (gradient boosting-ზე დაფუძნებული ხის ანსამბლის მოდელი)
  XGBoost (gradient boosting-ზე დაფუძნებული გადაწყვეტილების ხეების ანსამბლის მოდელი)
  Temporal Fusion Transformer (TFT) (ყურადღების მექანიზმსა და LSTM-ზე დაფუძნებული deep learning მოდელი       მრავალცვლადიანი დროითი რიგებისთვის)
  PatchTST (Transformer-ზე დაფუძნებული deep learning მოდელი დროითი რიგების პროგნოზირებისთვის)
  Prophet (ტრენდისა და სეზონურობის მოდელირებაზე ორიენტირებული სტატისტიკური forecasting მოდელი)

5. მოდელების ანალიზი

4.1 ARIMA (AutoARIMA)
როგორ მუშაობს
ARIMA წარმოადგენს კლასიკურ time-series მოდელს, რომელიც ეფუძნება სამ კომპონენტს:
  Autoregression (AR): წინა მნიშვნელობებზე დამოკიდებულება 
  Integrated (I): ტრენდის მოშორება differencing-ის გზით 
  Moving Average (MA): წარსული შეცდომების მოდელირება 
AutoARIMA ავტომატურად არჩევს საუკეთესო (p, d, q) პარამეტრებს თითოეული სერიისთვის.

როგორ გამოვიყენეთ
პროექტში ARIMA გამოყენებულ იქნა როგორც baseline მოდელი. თითოეული Store-Dept სერია დამუშავდა ცალ-ცალკე და მოდელმა გააკეთა 12 კვირიანი პროგნოზი validation პერიოდზე.

შედეგი და შეფასება
  ARIMA-მ აჩვენა სუსტი შედეგი:
  მაღალი შეცდომა (MAE, RMSE) 
  ძალიან მაღალი MAPE 
  baseline-ზე უარესი შედეგი 

ანალიზი
მთავარი პრობლემა იყო ის, რომ ARIMA:
  არ იყენებს გარე ფაქტორებს (promotions, holidays) 
  ვერ იჭერს არაწრფივ ქცევებს 
  სუსტია ძლიერი seasonal spikes-ის დროს 

4.2 SARIMA (Seasonal ARIMA)
როგორ მუშაობს
SARIMA წარმოადგენს ARIMA-ს გაფართოებულ ვერსიას, რომელიც დამატებით ითვალისწინებს სეზონურ კომპონენტს.
მოდელი მოიცავს:
  AR 
  I 
  MA 
  Seasonal component (წლიური/კვირეული განმეორებადობა) 

როგორ გამოვიყენეთ
მოდელში განისაზღვრა:
  seasonal=True 
  season_length=52 (კვირეული სეზონურობა) 
ეს საშუალებას აძლევს მოდელს დაიჭიროს yearly patterns, რაც Walmart-ის მონაცემებისთვის კრიტიკულია.

შედეგი და შეფასება
SARIMA-მ ARIMA-სთან შედარებით უკეთესი სეზონური სტრუქტურა დაიჭირა, თუმცა:
  შეცდომა კვლავ მაღალი დარჩა 
  extreme spikes ვერ დაიჭირა სრულად 

ანალიზი
SARIMA უკეთ მუშაობს სეზონურ მონაცემებზე, თუმცა მაინც რჩება ლინეარული მოდელი და ვერ ადაპტირდება რთულ არაწრფივ სტრუქტურებზე.

4.3 DLinear
როგორ მუშაობს
DLinear არის თანამედროვე deep learning forecasting მოდელი, რომელიც მონაცემს შლის ორ ნაწილად:
  Trend component 
  Seasonal component 
შემდეგ თითოეულზე იყენებს ცალკე linear layer-ს.

როგორ გამოვიყენეთ
მოდელი გაწვრთნილია:
  1 წლის lookback window-ზე 
  optimized loss function (MAPE/MAE) 
  MLflow tracking-ით 

გაუმჯობესებები
  ჰიპერპარამეტრების tuning 
  სწორი input window არჩევა 
  loss function-ის შეცვლა პროცენტულ შეცდომაზე ფოკუსირებით 

შედეგი
  DLinear-მ მნიშვნელოვნად გააუმჯობესა შედეგი:
  უკეთესი generalization 
  ნაკლები error ვიდრე ARIMA/SARIMA 
  სტაბილური პროგნოზი 

ანალიზი
მიუხედავად იმისა, რომ მოდელი ლინეარულია, decomposition მიდგომამ მას მისცა ძლიერი უნარი სეზონურობის და ტრენდის დაჭერისთვის.
თუმცა:
  ვერ ასახავს ძალიან არაწრფივ პიკებს სრულად 

4.4 N-BEATS
როგორ მუშაობს
N-BEATS არის ღრმა ნეირონული ქსელი, რომელიც სპეციალურად შექმნილია time-series forecasting-ისთვის.
მოდელი:
  არ საჭიროებს feature engineering-ს 
  ავტომატურად სწავლობს pattern-ებს 
  იყენებს residual blocks სტრუქტურას 
  შეუძლია trend და seasonality decomposition 

როგორ გამოვიყენეთ
მოდელი გავუშვით ორ რეჟიმში:
  baseline configuration 
  tuned configuration (loss, stack types, window size) 
დამატებით:
  გამოიყენება MLflow tracking 
  train/validation split დროითი წესით 

შედეგი
  N-BEATS-მ აჩვენა საუკეთესო შედეგები:
  უკეთესი capture of spikes 
  უკეთესი generalization 
  მაღალი სიზუსტე რთულ პერიოდებში (holiday weeks) 
4.5 LightGBM

ვცადე სხვადასხვა პრეპოცესინგი და feautre engineering. პრეპროცესინგმა შედეგი არ მიჩვენა, თUმცა featire engineering ში კარგი შედეგი მივიღე:
val_mae 1798.8251654108437
val_rmse 3995.1429063652927
train_rmse 2944.517143071157
train_mae 1528.289972178497
ოდნავი overfitting არის, თუმცა მასზე დაყრდნობით შეიძლება უკეთესი შედეგის მიღება. 
შემდომ ჰიპერპარამეტრების ცვლა დავიწყე რანდომ სეარჩით რამდენიმე გაშვების შედეგად საბოლოოდ ასეთი შედეგი მივიღე:
val_mae 1451.6190499308866
train_mae 1279.04622708597
val_rmse 2902.3785464615953
train_rmse 2442.61678054918

4.6 XGboost
გამოტოვებული შედეგები ვცადე სხვადასხვანაირად ჩამენაცვლებინა, თუმცა ყველაზე ერთნაირი შედეგი გამომივიდა. შემდგომ კორეალაციის ფილტრი ვცადე და ასევე არანაირი გაუმჯობესება არ გამოვიდა.საბოლოოდ ჯიპერპარამეტრებიამდე მივედი სადაც რანდომ სეარჩი გამოვიყენე და საბოლოოდ mae 1703.3253393029142 გამომივიდა და rmse 3904.1309411390666, რაც საუკეთესო შედეგი არ იყო სხვა მოდელებთან შედარებით.

4.7 Temporal Fusion Transformer (TFT)
დასაწყისში ვაპირებდით ამ მოდელზე დატრენინგებას, რადგან უფრო კომპლექსური იყო ვიდრე PatchTST, თუმცა პირველ გაშვება აღმოჩნდა 4-5 საათიანი. ვაპირებდი ღამით დატოვებას, თუმცა მე-7ზე ეპოქაზე გაიჭედა და საკმაოდ დიდი ხანი არაფერს შვრებოდა, რის გამოც მომიწია დაბრეიკება. შევარჩიე უფრო მარტივი ვარიანტი, თუმცა მე-5 ეპოქის მერე ისევ გაიჭედა და არაფერს აკეთებდა, ამიტომ გადავწყვიტე, რომ მიმეტოვებინა ეს მოდელი და გადავსულვიყავი PatchTST

4.8 PatchTST
ვცადე სხვადასხვა პარამეტრებზე ტრენინგი:
Hidden size: {64, 128, 256, 512}
Attention heads: {4, 8, 16}
Encoder layers: {1, 2, 3}
Learning rate: {0.01, 0.001, 0.0005}
Batch size: {32, 64, 128}
Maximum training steps: {500, 1000, 1500}

და საუკეთესო შედეგი მივიღე ამ პარამეტრებით:

Input size: 78
Patch length: 8
Stride: 4
Hidden size: 128
Number of attention heads: 4
Encoder layers: 1
თუმცა საუკეთესო შედეგი არ მიმიღია. ასევე ცდილობდი მეკონტროლებინდა loss ეპოქებს შორის რომ არ ყოფილიყო overfitting 

4.9 Prophet
ყველაზე ცუდი შედეგი ამ მოდელში მივიღე. აშკარა underfitting და უკეთეს შედეგამდე ვერ მივაღწიე. 
baseline:
yearly_seasonality=True
weekly_seasonality=True
daily_seasonality=False
seasonality_mode="multiplicative"
შემდგომ დავამატე seasonality და ასევე regressors.
შეზღუდული სისუსტე ჰქონდა ამ მოდელს. სხვადასხვა დამატებებმა აშკარა შედეგი იყონიეს, თუმცა სხვა მოდელებში ბევრად უკეთესი შედეგი იყო მიღებული

ანალიზი
N-BEATS არის ყველაზე ძლიერი მოდელი ამ პროექტში, რადგან:
  ის არ არის შეზღუდული ლინეარულობით 
  უკეთ იჭერს non-linear patterns-ს 
  ადაპტირდება complex seasonality-ზე 

5. მოდელების შედარებითი ანალიზი
Model    | Strengths                                               | Weaknesses
ARIMA    | მარტივი, სწრაფი, interpretable                          | ვერ ხედავს external factors და non-linearity
SARIMA   | სეზონურობის capture                                     | მაინც linear model
DLinear  | სწრაფი deep learning, strong baseline                   | limited non-linearity
N-BEATS  | ყველაზე ძლიერი forecasting capability                   | computationally heavy
LGBM     | სწრაფი, მაღალი accuracy, მუშაობს features-თან	         | საჭიროებს feature engineering-ს
PatchTST | ძლიერი Transformer forecasting, long-term dependencies	 | computationally expensive
Prophet	 | მარტივი, კარგი trend და seasonality capture	           | რთული patterns-ის სწავლა უჭირს
XGBoost	 | ძლიერი nonlinear model, მაღალი accuracy	               | საჭიროებს lag features-ს
TFT	     | ძლიერი multivariate forecasting, attention mechanism	   | ძალიან რთული და მძიმე მოდელია


7. საერთო დასკვნა
პროექტმა აჩვენა, რომ Walmart-ის მსგავსი მონაცემებისთვის:
კლასიკური სტატისტიკური მოდელები (ARIMA, SARIMA) შეზღუდულია რთული pattern-ებისა და external factors-ის დამუშავებაში.
Tree-based მოდელები (LightGBM, XGBoost) აჩვენებენ კარგ შედეგებს feature engineering-ის გამოყენებით, განსაკუთრებით tabular მონაცემებზე.
Deep learning მოდელები (DLinear, N-BEATS, PatchTST) უკეთ უმკლავდებიან complex temporal patterns-ს და გრძელვადიან დამოკიდებულებებს.
Prophet ეფექტურია trend და seasonality-ის მარტივი შემთხვევებისთვის, თუმცა რთულ მონაცემებზე ნაკლებად მოქნილია.
საბოლოოდ, საუკეთესო შედეგი მიღწეულ იქნა LGBM მოდელით, რომელმაც ყველაზე ეფექტურად დაიჭირა როგორც სეზონურობა, ასევე არაწრფივი ცვლილებები.

mlflow:
https://dagshub.com/tsarc21/Walmart-Recruiting---Store-Sales-Forecasting.mlflow/#/experiments
