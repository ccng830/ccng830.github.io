---
layout:     post
title:      "串接天氣API，開發一款根據天氣狀況實時切換背景的天氣App (2)"
date:       2020-03-07
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - API
---
## StoryBoard設置

如圖所示：
<img src="/img/weatherApp/6.png" width="100%">
首先插入Navigation Bar：Editor -> Embed In -> Navigation Bar
右邊勾選 Prefers Large Titles以及Bar Tint設為透明
最底層拉出一個ImageView，當然亦可選擇直接使用View處理background
右上角拉出一個Bar Button
中間拉出3個 UILabel，字體大小按自己喜歡設置
下方拉出一個UITable View，如以前教學，為TableView 拉datasource 及 delegate。
由於需從網絡上抓取資料，因此記得設置info.plist。

具體可參考下面的Link。
><a href="https://ccng830.github.io/2020/03/03/Google-Spreadsheet-JSON-&-imgur(2)/">利用Google Spreadsheet JSON & Imgur 開發App (2)</a>

## 完整代碼

本次作品為增強視覺效果，所以背景利用gif圖片，而播放gif 圖片參考了下面連結
><a href="https://github.com/kiritmodi2702/GIF-Swift/blob/master/GIF-Swift/iOSDevCenters%2BGIF.swift">GIF-Swift/GIF-Swift/iOSDevCenters+GIF.swift</a>

而我時前亦下載好一此較好的gif，放到本次的專案內
<img src="/img/weatherApp/7.png" width="100%">

上述的檔案下載後，直接拖到自己的專案中 / 開新Swift 檔，將code copy上去
<img src="/img/weatherApp/8.png" width="100%">

播放gif的使用方法是：

    let gif = UIImage.gifImageWithName(bg_name)
    self.bgImg.image = gif

直接上代碼，上完代碼後再解釋：

    import Foundation
    let id_list = ["200","201","202","210","211","212","221","230","231","232","300","301","302","310","311","312","313","314","321","500","501","502","503","504","511","520","521","522","531","600","601","602","611","612","613","615","616","620","621","622","701","711","721","731","741","751","761","762","771","781","800","801","802","803","804"]

    let weatherDescription_zh = ["雷陣雨","雷雨","雷暴雨","雷暴","雷暴","強雷暴","強雷暴","雷陣雨","雷雨","雷暴雨","毛毛雨","毛毛雨","毛毛雨","毛毛雨","毛毛雨","細雨","陣雨或細雨","陣雨或細雨","細雨","小雨","雨","大雨","暴雨","豪雨","凍雨","小陣雨","陣雨","強陣雨","強陣雨","薄雪","雪","大雪","霰","細雨夾雪","雨夾雪","小雨夾雪","雨夾雪","陣雪","雪","暴雪","薄霧","煙霞","霧霾","沙塵","霧","沙","塵","火山灰","狂風","龍卷風","天晴","天色明朗","多雲","天陰","密雲"]

    struct WeatherNow: Codable {
        let name: String
        let weather: [Weather]
        let main: Main
        let sys: Sys
    }

    struct Weather: Codable {
        let main: String
        let description: String
        let id: Int
    }

    struct Main: Codable {
        let temp: Double
        let temp_min: Double
        let temp_max: Double
        let feels_like: Double
        let humidity: Int
    }

    struct Sys: Codable {
        let sunrise: Double
        let sunset: Double
    }

我新增了一個Swift File用以管理上述變數，如之前分析JSON檔般，宣告對應的struct，由於抓下來的description為英文，我稍稍作了簡略的翻譯，將其存於weatherDescription_zh的list中，而它對應的ID我存於id_list之中。

ViewController.swift中：先上完整代碼，然後再分析，

    import UIKit

    class ViewController: UIViewController, UITableViewDataSource, UITableViewDelegate {
        
        var table_list = [String]()
        var table_img = ["clock","cloud.drizzle","thermometer","sunrise","sunset.fill"]
        @IBOutlet weak var myTableView: UITableView!
        func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return table_list.count
        }
        
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let Cell = UITableViewCell(style: UITableViewCell.CellStyle.default, reuseIdentifier: "Cell")
            tableView.rowHeight = 50
            tableView.backgroundColor = UIColor.clear
            Cell.backgroundColor = UIColor.clear
            Cell.selectionStyle = .none
            Cell.textLabel?.textColor = UIColor.white
            Cell.textLabel?.text = table_list[indexPath.row]
            Cell.imageView?.image = UIImage(systemName: table_img[indexPath.row])
            Cell.imageView?.tintColor = UIColor.white
            return Cell
        }
        
        func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
            if indexPath.row == 3 {
                ChangeBg("Sunrise")
                self.weather_description.text = "不要錯過日出！"
                DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 11) {
                    self.getWeather()
                }
            }
            if indexPath.row == 4 {
                ChangeBg("Sunset")
                self.weather_description.text = "不要錯過日落!"
                DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 10.5) {
                    self.getWeather()
                }
            }
        }
        
        var weatherDescription1,weatherDescription2: String?
        var ID: Int?
        var refreshControl:UIRefreshControl!
        var timer: Timer?
        @IBOutlet weak var bgImg: UIImageView!
        @IBOutlet weak var temp_now: UILabel!
        @IBOutlet weak var temp_MinToMax: UILabel!
        @IBOutlet weak var weather_description: UILabel!
        @IBOutlet weak var TestModeButton: UIBarButtonItem!
        
        override func viewWillAppear(_ animated: Bool) {
            super.viewWillAppear(true)
            TestModeButton.tintColor = UIColor.white
            getWeather()
        }
        
        override func viewDidLoad() {
            super.viewDidLoad()
            refreshControl = UIRefreshControl()
            refreshControl.tintColor = UIColor.white
            myTableView.addSubview(refreshControl)
            refreshControl.addTarget(self, action: #selector(Refreshing), for: UIControl.Event.valueChanged)
            self.timer = Timer.scheduledTimer(timeInterval: 900, target: self, selector: #selector(ViewController.getWeather), userInfo: nil, repeats: true) //每15分鐘更新一次
        }
        
        //MARK:--- Test bg Button
        @IBAction func ChangeBg(_ sender: UIBarButtonItem) {
            let bg_list = ["Cloudy","Cloudy2","Cloud night","Night","Rain","Rain2","Sunny","Foggy","Snow","Thunder"]
            let controller = UIAlertController(title: nil, message: nil, preferredStyle: .actionSheet)
            for item in bg_list {
                let action = UIAlertAction(title: item, style: .default) { (_) in
                    self.ChangeBg(item)
                }
                controller.addAction(action)
            }
            let cancelAction = UIAlertAction(title: "取消", style: .cancel, handler: nil)
            controller.addAction(cancelAction)
            present(controller, animated: true, completion: nil)
        }
        
        //MARK:--- Change bg
        func ChangeBg(_ bg_name: String) {
            let gif = UIImage.gifImageWithName(bg_name)
            self.bgImg.image = gif
        }
        
        func ChangeBgViaApi(_ ID: String, _ sunrise: Double, _ sunset: Double) {
            var bg_name = "Cloudy2"
            let state = getState(sunrise, sunset)
            let ID_1 = ID.prefix(1) //取得ID首位
            
            if ID_1 == "2" {
                bg_name = "Thunder"
            } else if ID_1 == "3" || ID_1 == "5" {
                bg_name = "Rain2"
            } else if ID_1 == "6" {
                bg_name = "Snow"
            } else if ID_1 == "7"{
                bg_name = "Foggy"
            } else if ID_1 == "8" {
                if ID == "800"{
                    if state == "晚上" {
                        bg_name = "Night"
                    } else {
                        bg_name = "Sunny"
                    }
                } else {
                    if state == "早上" {
                        bg_name = "Cloudy2"
                    } else if state == "晚上" {
                        bg_name = "Cloud night"
                    } else {
                        bg_name = "Cloudy"
                    }
                }
            }
            let gif = UIImage.gifImageWithName(bg_name)
            self.bgImg.image = gif
        }
        
        func getState(_ Rise: Double, _ Set: Double) -> String{
            let formmat = "HHmmss"
            let formmat1 = DateFormatter()
            var state = ""
            formmat1.dateFormat = formmat
            let now = Int(formmat1.string(from: Date()))
            let sunrise = Int(formmat1.string(from: Date(timeIntervalSince1970: Rise)))
            let sunset = Int(formmat1.string(from: Date(timeIntervalSince1970: Set)))
            
            if sunset!-20000 < now! && now! < sunset! {
                state = "傍晚"
            } else if now! < sunrise! || sunset! < now! {
                state = "晚上"
            } else {
                state = "早上"
            }
            return state
        }
        
        //MARK:--- setup label
        func SetLabel(_ temp_now: String, _ temp_min: String, _ temp_max: String, _ weather_description: String){
            self.temp_now.text = temp_now
            self.temp_MinToMax.text = temp_min + " - " + temp_max
            self.weather_description.text = weather_description
        }
        
        //MARK:--- pull to refresh
        @objc func Refreshing() {
            self.refreshControl.endRefreshing()
            self.getWeather()
        }
        
        //MARK:--- Weather API
        func Temp(_ num: Double) -> String {
            return Int(floor(num - 273.15)).description+"℃"
        }

        func transTime(_ num: Double) -> String {
            let date = Date(timeIntervalSince1970: num)
            let formmat = "HH:mm:ss"
            let formmat1 = DateFormatter()
            formmat1.dateFormat = formmat
            let dateString = formmat1.string(from: date)
            return dateString
        }
        
        func getUpdateTime() -> String {
            let date = Date()
            let formmat = "MM/dd HH:mm:ss"
            let formmat1 = DateFormatter()
            formmat1.dateFormat = formmat
            let dateString = formmat1.string(from: date)
            return dateString
        }

        @objc func getWeather() {
            let address = "http://api.openweathermap.org/data/2.5/weather?q=macao&appid=84a74ead2518982332a6bb825b67bfb8"
            if let url = URL(string: address) {
                // GET
                URLSession.shared.dataTask(with: url) { (data, response, error) in
                    if let error = error {
                        print("Error: \(error.localizedDescription)")
                    } else if let response = response as? HTTPURLResponse,let data = data {
                        print("Status code: \(response.statusCode)")
                        let decoder = JSONDecoder()
                        
                        if let weatherDetail = try? decoder.decode(WeatherNow.self, from: data) {
                            DispatchQueue.main.async{
                                //title setup
                                let place = weatherDetail.name //地方
                                self.navigationItem.title = "澳門"
                                
                                //label setup
                                for item in weatherDetail.weather {
                                    self.weatherDescription1 = item.main //天氣狀況
                                    self.weatherDescription2 = item.description //詳細天氣狀況
                                    self.ID = item.id
                                }
                                
                                let index = id_list.firstIndex(of: self.ID!.description)
                                self.weatherDescription2 = weatherDescription_zh[index!]
                                
                                let temp = self.Temp(weatherDetail.main.temp) //現時溫度
                                let temp_max = self.Temp(weatherDetail.main.temp_max) //最高溫
                                let temp_min = self.Temp(weatherDetail.main.temp_min) //最低溫
                                
                                self.SetLabel(temp, temp_min, temp_max, self.weatherDescription2!)
                                
                                //table view setup
                                let updatetime = self.getUpdateTime()
                                let feel_like = self.Temp(weatherDetail.main.feels_like) //體感溫度
                                let hum = weatherDetail.main.humidity.description + "%" //濕度
                                let sunrise = self.transTime(weatherDetail.sys.sunrise) //日出時間
                                let sunset = self.transTime(weatherDetail.sys.sunset) //日落時間
                                
                                //change bg
                                self.ChangeBgViaApi(self.ID!.description, weatherDetail.sys.sunrise, weatherDetail.sys.sunset)
                                
                                //table view setup
                                self.table_list = []
                                self.table_list.append("更新時間：" + updatetime)
                                self.table_list.append("相對濕度：" + hum)
                                self.table_list.append("體感溫度：" + feel_like)
                                self.table_list.append("日出時間：" + sunrise)
                                self.table_list.append("日落時間：" + sunset)
                                self.myTableView.reloadData()
                            }
                        }
                    }
                }.resume()
            } else {
                print("Invalid URL.")
            }
        }
    }

## 分段解析
## TableView設定部分
一開始是TableView的一些設定

    var table_list = [String]()
    var table_img = ["clock","cloud.drizzle","thermometer","sunrise","sunset.fill"]
    @IBOutlet weak var myTableView: UITableView!
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return table_list.count
    }

很好理解，返回table_list中的元素個數以作為tableView的Rows

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let Cell = UITableViewCell(style: UITableViewCell.CellStyle.default, reuseIdentifier: "Cell")
        tableView.rowHeight = 50
        tableView.backgroundColor = UIColor.clear
        Cell.backgroundColor = UIColor.clear
        Cell.selectionStyle = .none
        Cell.textLabel?.textColor = UIColor.white
        Cell.textLabel?.text = table_list[indexPath.row]
        Cell.imageView?.image = UIImage(systemName: table_img[indexPath.row])
        Cell.imageView?.tintColor = UIColor.white
        return Cell
    }

設定了row的高度為50，tableView及Cell的背景顏色均為無色，selectionStyle為none，即當用家點選對應的rows時不會出現反色，textColor為白色，Cell的image，即每一個rows左邊的圖片我直接使用了system的內部圖片，分別為table_img = ["clock","cloud.drizzle","thermometer","sunrise","sunset.fill"]，並對它為著色為白色，最後返回Cell的設定。


    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        if indexPath.row == 3 {
            ChangeBg("Sunrise")
            self.weather_description.text = "不要錯過日出！"
            DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 11) {
                self.getWeather()
            }
        }
        if indexPath.row == 4 {
            ChangeBg("Sunset")
            self.weather_description.text = "不要錯過日落!"
            DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 10.5) {
                self.getWeather()
            }
        }
    }
    
小小的加入一個彩蛋，當用家按下日出/日落的rows，背景會變為日出/日落，播放約10秒後便重新加載現時的天氣情況。

## Bar Button 部分
    @IBAction func ChangeBg(_ sender: UIBarButtonItem) {
        let bg_list = ["Cloudy","Cloudy2","Cloud night","Night","Rain","Rain2","Sunny","Foggy","Snow","Thunder"]
        let controller = UIAlertController(title: nil, message: nil, preferredStyle: .actionSheet)
        for item in bg_list {
            let action = UIAlertAction(title: item, style: .default) { (_) in
                self.ChangeBg(item)
            }
            controller.addAction(action)
        }
        let cancelAction = UIAlertAction(title: "取消", style: .cancel, handler: nil)
        controller.addAction(cancelAction)
        present(controller, animated: true, completion: nil)
    }
    
    func ChangeBg(_ bg_name: String) {
        let gif = UIImage.gifImageWithName(bg_name)
        self.bgImg.image = gif
    }

這個按鈕可有可無，用於測試背景的變化用。
首先，bg_list存取了我所加入的gif檔的名稱，當按下按鈕後，產新一個UIAlertController，preferredStyle: .actionSheet，利用for將bg_list的內容添加到選單中，最後額外加一個取消按鈕。

## 改變背景

    func ChangeBgViaApi(_ ID: String, _ sunrise: Double, _ sunset: Double) {
        var bg_name = "Cloudy2"
        let state = getState(sunrise, sunset)
        let ID_1 = ID.prefix(1) //取得ID首位
        
        if ID_1 == "2" {
            bg_name = "Thunder"
        } else if ID_1 == "3" || ID_1 == "5" {
            bg_name = "Rain2"
        } else if ID_1 == "6" {
            bg_name = "Snow"
        } else if ID_1 == "7"{
            bg_name = "Foggy"
        } else if ID_1 == "8" {
            if ID == "800"{
                if state == "晚上" {
                    bg_name = "Night"
                } else {
                    bg_name = "Sunny"
                }
            } else {
                if state == "早上" {
                    bg_name = "Cloudy2"
                } else if state == "晚上" {
                    bg_name = "Cloud night"
                } else {
                    bg_name = "Cloudy"
                }
            }
        }
        let gif = UIImage.gifImageWithName(bg_name)
        self.bgImg.image = gif
    }

    func getState(_ Rise: Double, _ Set: Double) -> String{
        let formmat = "HHmmss"
        let formmat1 = DateFormatter()
        var state = ""
        formmat1.dateFormat = formmat
        let now = Int(formmat1.string(from: Date()))
        let sunrise = Int(formmat1.string(from: Date(timeIntervalSince1970: Rise)))
        let sunset = Int(formmat1.string(from: Date(timeIntervalSince1970: Set)))
        
        if sunset!-20000 < now! && now! < sunset! {
            state = "傍晚"
        } else if now! < sunrise! || sunset! < now! {
            state = "晚上"
        } else {
            state = "早上"
        }
        return state
    }

ChangeBgViaApi(_ ID: String, _ sunrise: Double, _ sunset: Double) 要求輸入ID，日出及日落的unix時間戳，如之前所說，ID_1取得ID的第一位，判斷天氣狀況，由於800是晴天，而8XX是有雲，因此要再判斷一次。
同時有些背景我準備了早上，晚上，及傍晚的版本，因次getState(_ Rise: Double, _ Set: Double) 是比較日出日落時間得到現時是早上或晚上。

## 串接API

    func Temp(_ num: Double) -> String {
        return Int(floor(num - 273.15)).description+"℃"
    }

    func transTime(_ num: Double) -> String {
        let date = Date(timeIntervalSince1970: num)
        let formmat = "HH:mm:ss"
        let formmat1 = DateFormatter()
        formmat1.dateFormat = formmat
        let dateString = formmat1.string(from: date)
        return dateString
    }

    func getUpdateTime() -> String {
        let date = Date()
        let formmat = "MM/dd HH:mm:ss"
        let formmat1 = DateFormatter()
        formmat1.dateFormat = formmat
        let dateString = formmat1.string(from: date)
        return dateString
    }

    @objc func getWeather() {
        let address = "http://api.openweathermap.org/data/2.5/weather?q=macao&appid=84a74ead2518982332a6bb825b67bfb8"
        if let url = URL(string: address) {
            // GET
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                if let error = error {
                    print("Error: \(error.localizedDescription)")
                } else if let response = response as? HTTPURLResponse,let data = data {
                    print("Status code: \(response.statusCode)")
                    let decoder = JSONDecoder()
                    
                    if let weatherDetail = try? decoder.decode(WeatherNow.self, from: data) {
                        DispatchQueue.main.async{
                            //title setup
                            let place = weatherDetail.name //地方
                            self.navigationItem.title = "澳門"
                            
                            //label setup
                            for item in weatherDetail.weather {
                                self.weatherDescription1 = item.main //天氣狀況
                                self.weatherDescription2 = item.description //詳細天氣狀況
                                self.ID = item.id
                            }
                            
                            let index = id_list.firstIndex(of: self.ID!.description)
                            self.weatherDescription2 = weatherDescription_zh[index!]
                            
                            let temp = self.Temp(weatherDetail.main.temp) //現時溫度
                            let temp_max = self.Temp(weatherDetail.main.temp_max) //最高溫
                            let temp_min = self.Temp(weatherDetail.main.temp_min) //最低溫
                            
                            self.SetLabel(temp, temp_min, temp_max, self.weatherDescription2!)
                            
                            //table view setup
                            let updatetime = self.getUpdateTime()
                            let feel_like = self.Temp(weatherDetail.main.feels_like) //體感溫度
                            let hum = weatherDetail.main.humidity.description + "%" //濕度
                            let sunrise = self.transTime(weatherDetail.sys.sunrise) //日出時間
                            let sunset = self.transTime(weatherDetail.sys.sunset) //日落時間
                            
                            //change bg
                            self.ChangeBgViaApi(self.ID!.description, weatherDetail.sys.sunrise, weatherDetail.sys.sunset)
                            
                            //table view setup
                            self.table_list = []
                            self.table_list.append("更新時間：" + updatetime)
                            self.table_list.append("相對濕度：" + hum)
                            self.table_list.append("體感溫度：" + feel_like)
                            self.table_list.append("日出時間：" + sunrise)
                            self.table_list.append("日落時間：" + sunset)
                            self.myTableView.reloadData()
                        }
                    }
                }
            }.resume()
        } else {
            print("Invalid URL.")
        }
    }

由於得到的溫度是開氏溫標，因此func Temp用來轉換溫度。而得到的時間是unix時間戳，所以func transTime用以轉換用。getWeather()用來取得API的資料。並且將資料傳入到對應的位置，並且利用reloadData()更新TableView。

## UIRefreshControl( )

    var refreshControl:UIRefreshControl!
    var timer: Timer?
    override func viewDidLoad() {
        super.viewDidLoad()
        refreshControl = UIRefreshControl()
        refreshControl.tintColor = UIColor.white
        myTableView.addSubview(refreshControl)
        refreshControl.addTarget(self, action: #selector(Refreshing), for: UIControl.Event.valueChanged)
        self.timer = Timer.scheduledTimer(timeInterval: 900, target: self, selector: #selector(ViewController.getWeather), userInfo: nil, repeats: true) //每15分鐘更新一次
    }

    @objc func Refreshing() {
        self.refreshControl.endRefreshing()
        self.getWeather()
    }
    
為TableView.addSubview(refreshControl)添加refreshControl，加入下拉功能，下拉時調動self.getWeather()功能以更新變數。利用timer 定時每15分鐘更新一次。
