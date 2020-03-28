---
layout:     post
title:      "利用Google Spreadsheet JSON & Imgur 開發App (2)"
date:       2020-03-03
author:     "cc"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - API
---
## 簡介

開發前我們先觀看一下成品，
<img src="/img/googleJson2/1.gif" width="60%">

可以看到該App實際上就是一個TableView，內裡的Cell顯示的資料為上一節中所抓取的寵物小精靈資料。

## API串接及JSON解析(抓取資料)

開一個新專案，命名為API_test，先上操作及完整代碼

首先加入Navigation Controller，將title 改名為Pokemon
<img src="/img/googleJson2/2.gif" width="100%">

然後拖出TableView，並按著command拖動，點選dataSource 及 delegate。
<img src="/img/googleJson2/3.gif" width="100%">

最後Prototype cells新增1，點選cell將Identify改名為Cell
<img src="/img/googleJson2/4.gif" width="100%">

關掉Xcode，打開terminal，cd到專案file 並輸入pod init
<img src="/img/googleJson2/img1.png" width="100%">

專案File下多出一個Podfile，打開Podfile，輸入
pod 'SDWebImage', :modular_headers => true
並且要註釋掉 #use_frameworks!
<img src="/img/googleJson2/img2.png" width="100%">

再到terminal，輸入pod install，安裝後，打開下圖檔案
<img src="/img/googleJson2/img3.png" width="100%">

進入info.plist，如下圖加入
<img src="/img/googleJson2/5.gif" width="100%">

    import UIKit
    import SDWebImage

    class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource  {
        
        var pokeDetail: [String] = []
        var url = [String]()
        @IBOutlet weak var myTableView: UITableView!
        
        func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return pokeDetail.count
        }
        
        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            tableView.rowHeight = 180
            let cell = UITableViewCell(style: UITableViewCell.CellStyle.default, reuseIdentifier: "Cell")
            cell.imageView?.sd_setImage(with: URL(string: url[indexPath.row]))
            cell.textLabel?.text = pokeDetail[indexPath.row]
            return cell
        }
        
        override func viewDidLoad() {
            super.viewDidLoad()
            getpokeData()
        }

        //MARK: --- Pokemon API
        struct PokeResults: Codable {
            let rows: [Poke]
        }

        struct Poke: Codable {
            let num: String
            let name: String
            let property: String
            let image: String
        }

        func getpokeData() {
            let address = "http://gsx2json.com/api?id=1ybZ6Kk1wIKIZnQ_nPtRNNO5iX8Q5U9pz8qWD2jrf1l8&columns=false"
            if let url = URL(string: address) {
                // GET
                URLSession.shared.dataTask(with: url) { (data, response, error) in
                    if let error = error {
                        print("Error: \(error.localizedDescription)")
                    } else if let response = response as? HTTPURLResponse,let data = data {
                        print("Status code: \(response.statusCode)")
                        let decoder = JSONDecoder()
                        
                        if let pokeResults = try? decoder.decode(PokeResults.self, from: data) {
                            DispatchQueue.main.async{
                                for poke in pokeResults.rows {
                                    self.pokeDetail.append(poke.num + " " + poke.name + " " + poke.property)
                                    self.url.append(poke.image)
                                }
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

import SDWebImage 是使用第三方插件，使用該插件可以簡單地透過URL抓取圖片。

pokeDetail及url兩個Array是用來儲存資料，拉出tableView的IBOutlet。

下面是tableView 的Rows數目，因些我們返回pokeDetail的個數作其Rows的數目。

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return pokeDetail.count
    }

下面則是tableViewCell的一些設定，設定每行高度為180，cell的imageView則是透過第三方插件SDWebImage的功能透過給予url去抓取圖片。

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        tableView.rowHeight = 180
        let cell = UITableViewCell(style: UITableViewCell.CellStyle.default, reuseIdentifier: "Cell")
        cell.imageView?.sd_setImage(with: URL(string: url[indexPath.row]))
        cell.textLabel?.text = pokeDetail[indexPath.row]
        return cell
    }

定義了兩個結構，分別為PokeResults及Poke。為什麼要定義兩個結構，原因是由於上一節所得到的JSON（如下圖），內裡包含了rows，而rows是一個字串，內裡又包含了num，name，property，image等，因此為了對應該JSON的格式就需要對應定義出兩個struct，第一個struct(PokeResults)之後用來抓取JSON的rows，而第二個struct(Poke)則是抓取每個rows中的元素(num, name...)。

    struct PokeResults: Codable {
        let rows: [Poke]
    }

    struct Poke: Codable {
        let num: String
        let name: String
        let property: String
        let image: String
    }

<img src="/img/googleJson/img6.png" width="100%">

下面定義了func getpokeData()，address是上節我們得到的API，利用for poke in pokeResults.rows分別將num, name, property的資料存到pokeDetail中，而image則存到url中，然後更新TableView的數據。

    func getpokeData() {
        let address = "http://gsx2json.com/api?id=1ybZ6Kk1wIKIZnQ_nPtRNNO5iX8Q5U9pz8qWD2jrf1l8&columns=false"
        if let url = URL(string: address) {
            // GET
            URLSession.shared.dataTask(with: url) { (data, response, error) in
                if let error = error {
                    print("Error: \(error.localizedDescription)")
                } else if let response = response as? HTTPURLResponse,let data = data {
                    print("Status code: \(response.statusCode)")
                    let decoder = JSONDecoder()
                    
                    if let pokeResults = try? decoder.decode(PokeResults.self, from: data) {
                        DispatchQueue.main.async{
                            for poke in pokeResults.rows {
                                self.pokeDetail.append(poke.num + " " + poke.name + " " + poke.property)
                                self.url.append(poke.image)
                            }
                            self.myTableView.reloadData()
                        }
                    }
                }
            }.resume()
        } else {
            print("Invalid URL.")
        }
    }
