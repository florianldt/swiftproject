# //
//  DancePlayViewController.swift
//  BoomClap
//
//  Created by 이규현 on 2020/01/30.
//  Copyright © 2020 이규현. All rights reserved.
//
import UIKit
import Alamofire
import AVFoundation

//2020.02.02 add
struct AVPlayerQueueBuilder {

    //
    static func from(_ urls: [URL]) -> [AVPlayerItem] {
        return urls.map { return AVPlayerItem(url: $0) }
    }
}

class DancePlayViewController: UIViewController {

    //전 화면에서 받은 데이터
    var noreceivedValueFromBeforeVC = ""
    var scopereceivedValueFromBeforeVC = ""
    var dancefilereceivedValueFromBeforeVC = ""
    var titlereceivedValueFromBeforeVC = ""
    var creatorreceivedValueFromBeforeVC = ""
    var textreceivedValueFromBeforeVC = ""
    var numberoflikereceivedValueFromBeforeVC = ""
    var checklikereceivedValueFromBeforeVC = ""
    
    //전역변수 설정
    var no: [String] = []
    var scope: [String] = []
    var title1: [String] = []
    var dancefile: [String]!
    var creator: [String] = []
    var text: [String] = []
    var numberoflike: [String] = []
    var checklike: [String] = []
    var downloadedData: [String]!
    
    //
    var isPlaying = false
    
    
    //
    var player: AVQueuePlayer = {
        var items = AVPlayerQueueBuilder.from(Settings.videoUrls as! [URL])
        var player = AVQueuePlayer(items: items)
        return player
    }()

    //
    var counter = 6

    //
    enum Settings {
        static let screenW: CGFloat = 375
        static let playerHeight: CGFloat = Settings.screenW * (9/16)

        //static let videoUrl = URL(string: "http://download.appboomclap.co.kr/ad/ad_1.mp4")!

        static var videoUrls = [
            //광고
            URL(string: "http://download.appboomclap.co.kr/ad/ad_1.mp4"),

            //댄스 영상
            //http://download.appboomclap.co.kr/content/content_301.mp4
            URL(string: "http://download.appboomclap.co.kr/content/\(self.dancefilereceivedValueFromBeforeVC).mp4"),
        ]
    }
    
    //
    let activityIndicatorView: UIActivityIndicatorView = {
        let aiv = UIActivityIndicatorView(style: .whiteLarge)
        aiv.isUserInteractionEnabled = true
        aiv.translatesAutoresizingMaskIntoConstraints = false
        aiv.startAnimating()
        return aiv
    }()
    
    //
    let playerView: UIView = {
        let view = UIView()
        view.translatesAutoresizingMaskIntoConstraints = false
        view.backgroundColor = .black
        return view
    }()
    
    //
    let progressView: UIView = {
        let prov = UIView()
        prov.translatesAutoresizingMaskIntoConstraints = false
        prov.backgroundColor = .blue
        return prov
    }()
    
    //
    lazy var pausePlayButton: UIButton = {
        let button = UIButton(type: .system)
        let image = UIImage(named: "pause")
        button.setImage(image, for: .normal)
        button.isUserInteractionEnabled = true
        button.translatesAutoresizingMaskIntoConstraints = false
        button.tintColor = .white
        button.isHidden = true
        button.addTarget(self, action: #selector(handlePause), for: .touchUpInside)
        return button
    }()
    
    //
    let videoSlider: UISlider = {
        let slider = UISlider()
        slider.translatesAutoresizingMaskIntoConstraints = false
        slider.minimumTrackTintColor = .red
        slider.isUserInteractionEnabled = true
        slider.maximumTrackTintColor = .white
        slider.setThumbImage(UIImage(named: "thumb"), for: .normal)
        slider.addTarget(self, action: #selector(handleSliderChange), for: .valueChanged)
        return slider
    }()
    
    //
    let currentTimeLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "00:00"
        label.isUserInteractionEnabled = true
        label.textColor = .black
        label.font = UIFont.boldSystemFont(ofSize: 13)
        return label
    }()
    
    //
    let videoLengthLabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "00:00"
        label.textColor = .black
        label.isUserInteractionEnabled = true
        label.font = UIFont.boldSystemFont(ofSize: 14)
        label.textAlignment = .right
        return label
    }()
    
    //
    var playerLayer: AVPlayerLayer = {
        let layer = AVPlayerLayer()
        layer.videoGravity = AVLayerVideoGravity(rawValue: AVLayerVideoGravity.resize.rawValue)
        layer.needsDisplayOnBoundsChange = true
        return layer
    }()
    
    //
    let titlelabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "title"
        label.isUserInteractionEnabled = true
        label.textColor = .white
        //label.font = UIFont.boldSystemFont(ofSize: 18)
        label.font = UIFont.systemFont(ofSize: 18)
        label.textColor = .black
        label.textAlignment = .right
        return label
    }()
    
    //
    let subtitlelabel: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "subtitle"
        label.isUserInteractionEnabled = true
        label.textColor = .white
        //label.font = UIFont.boldSystemFont(ofSize: 16)
        label.font = UIFont.systemFont(ofSize: 16)
        label.textColor = .black
        label.textAlignment = .right
        return label
    }()
    
    //
    let likenumber: UILabel = {
        let label = UILabel()
        label.translatesAutoresizingMaskIntoConstraints = false
        label.text = "999"
        label.textColor = .white
        label.isUserInteractionEnabled = true
        label.font = UIFont.boldSystemFont(ofSize: 16)
        label.textColor = .black
        label.textAlignment = .right
        return label
    }()
    
    //
    let likebutton: UIButton = {
        //let heart : UIImage = UIImage(named:"lecture_btn_s2_on")!
        //let no_heart : UIImage = UIImage(named:"lecture_btn_s2_off")!
        let button = UIButton()
        button.addTarget(self, action: #selector(pressed2), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        button.backgroundColor = .blue
        return button
    }()
    
    //
    let skipbutton: UIButton = {
       
        let button = UIButton()
        button.translatesAutoresizingMaskIntoConstraints = false
        //skipbutton.setTitle("A", for: .normal)
        button.backgroundColor = UIColor.red
        button.isHidden = true
        button.addTarget(self, action: #selector(skipbuttonAction), for: .touchUpInside)
        return button
    }()
    
    
    //
    override func viewDidLoad() {
        super.viewDidLoad()
        
       //getFullPath(fileName: dancefilereceivedValueFromBeforeVC)

        //
        //let videoUrl = [URL(string: "http://download.appboomclap.co.kr/ad/ad_1.mp4")!,
        //URL(string: "http://download.appboomclap.co.kr/content/\(self.dancefilereceivedValueFromBeforeVC).mp4")!]
        
        //let items = AVPlayerQueueBuilder.from(videoUrl)
        //let player = AVQueuePlayer(items: items)
        
        //let items = AVPlayerQueueBuilder.from(videoUrl)
        //let player = AVQueuePlayer(items: items)
        
        
        
//        //Rest API URL 요청(데이터 요청)
//        let _url = "http://www.appboomclap.co.kr:8080/BoomClap/Main"
//        let parameters: Parameters = [ "protocol": NetworkProtocol().GET_AD_REQ, ]
//        Alamofire.request(_url, method: .post, parameters: parameters).responseJSON { response in
//            if response.result.value != nil {
//
//            }
//
//            if let data = response.data, let utf8Text = String(data: data, encoding: .utf8) {
//                let utf8text = "\(utf8Text)"
//                var cnt = 0
//                var array : [String] = ["a","b","c","d","e","f","g","h"]
//
//                //
//                for utf8text in utf8text.components(separatedBy: .newlines) {
//                    array[cnt] = utf8text
//                    cnt += 1
//                }
//
//                //서버에서 받아온 데이터를 자른 후 배열에 담음
//                let c = array[2].split {$0 == "$"}.map(String.init)
//                for i in 0..<c.count {
//                    self.downloadedData = c[i].split {$0 == "#"}.map(String.init)
//                    self.no.append(self.downloadedData[0])
//                    self.scope.append(self.downloadedData[1])
//                    self.title1.append(self.downloadedData[2])
//                    self.dancefile.append(self.downloadedData[3])
//                    self.creator.append(self.downloadedData[4])
//                    self.text.append(self.downloadedData[5])
//                    self.numberoflike.append(self.downloadedData[6])
//                    self.checklike.append(self.downloadedData[7])
//                }
//            }
//        }
        
        
        
        //
        let timer = Timer.scheduledTimer(timeInterval: 1.0, target: self, selector: #selector(update), userInfo: nil, repeats: true)
        
        //
        print("요청결과 : \(noreceivedValueFromBeforeVC)")
        print("번호 : \(scopereceivedValueFromBeforeVC)")
        print("댄스파일 : \(dancefilereceivedValueFromBeforeVC)")
        
        titlelabel.text = titlereceivedValueFromBeforeVC
        print("제목 : \(titlereceivedValueFromBeforeVC)")
        
        subtitlelabel.text = creatorreceivedValueFromBeforeVC
        print("부제목 : \(creatorreceivedValueFromBeforeVC)")
        print("텍스트: \(textreceivedValueFromBeforeVC)")
        
        likenumber.text = numberoflikereceivedValueFromBeforeVC
        print("좋아요 수 : \(numberoflikereceivedValueFromBeforeVC)")
        print("좋아요 체크 : \(checklikereceivedValueFromBeforeVC)")
        
        //
        view.backgroundColor = .gray
        
        //
        navigationItem.title = "춤추기"
        navigationController?.navigationBar.isTranslucent = false
        
        //
        view.addSubview(playerView)
        view.addSubview(activityIndicatorView)
        view.addSubview(pausePlayButton)
        view.addSubview(progressView)
        
        //
        progressView.addSubview(videoLengthLabel)
        progressView.addSubview(currentTimeLabel)
        progressView.addSubview(videoSlider)
        
        //
        view.addSubview(titlelabel)
        view.addSubview(subtitlelabel)
        view.addSubview(likenumber)
        view.addSubview(likebutton)
        view.addSubview(skipbutton)
        
        //
        setupPlayerView()
        player.play()
        
        
        //
        let loginbutton = UIButton()
        loginbutton.setTitle("춤추기", for: .normal)
        loginbutton.setTitleColor(UIColor.white, for: .normal)
        loginbutton.backgroundColor = UIColor(red: 249/255, green: 138/255, blue: 170/255, alpha: 1.0)
        loginbutton.addTarget(self, action: #selector(self.pressed), for: .touchUpInside)
        self.view.addSubview(loginbutton)
        
        loginbutton.snp.makeConstraints { (make) in
            make.leading.equalTo(0)
            make.trailing.equalTo(0)
            make.height.equalTo(60)
            make.bottom.equalTo(self.view.safeAreaLayoutGuide.snp.bottom)
        }
    }
    
    //
    private func setupPlayerView() {
        
        //
        NSLayoutConstraint.activate([
            activityIndicatorView.centerXAnchor.constraint(equalTo: playerView.centerXAnchor),
            activityIndicatorView.centerYAnchor.constraint(equalTo: playerView.centerYAnchor),
        ])
        
        //일시정지 버튼
        NSLayoutConstraint.activate([
            pausePlayButton.centerXAnchor.constraint(equalTo: playerView.centerXAnchor),
            pausePlayButton.centerYAnchor.constraint(equalTo: playerView.centerYAnchor),
            pausePlayButton.widthAnchor.constraint(equalToConstant: 50),
            pausePlayButton.heightAnchor.constraint(equalToConstant: 50),
        ])
        
        //
        NSLayoutConstraint.activate([
            playerView.topAnchor.constraint(equalTo: view.topAnchor),
            playerView.leftAnchor.constraint(equalTo: view.leftAnchor),
            playerView.rightAnchor.constraint(equalTo: view.rightAnchor),
            playerView.heightAnchor.constraint(equalToConstant: Settings.playerHeight),
        ])
        
        //
        NSLayoutConstraint.activate([
            progressView.leftAnchor.constraint(equalTo: playerView.leftAnchor),
            progressView.bottomAnchor.constraint(equalTo: playerView.bottomAnchor, constant: 24),
            progressView.rightAnchor.constraint(equalTo: view.rightAnchor),
            progressView.widthAnchor.constraint(equalToConstant: 60),
            progressView.heightAnchor.constraint(equalToConstant: 40),
        ])
        
        //총 시간
        NSLayoutConstraint.activate([
            videoLengthLabel.rightAnchor.constraint(equalTo: progressView.rightAnchor, constant: -8),
            videoLengthLabel.bottomAnchor.constraint(equalTo: progressView.bottomAnchor, constant: -8),
            videoLengthLabel.widthAnchor.constraint(equalToConstant: 60),
            videoLengthLabel.heightAnchor.constraint(equalToConstant: 24),
        ])
        
        //이동시간
        NSLayoutConstraint.activate([
            currentTimeLabel.leftAnchor.constraint(equalTo: progressView.leftAnchor, constant: 8),
            currentTimeLabel.bottomAnchor.constraint(equalTo: progressView.bottomAnchor, constant: -8),
            currentTimeLabel.widthAnchor.constraint(equalToConstant: 60),
            currentTimeLabel.heightAnchor.constraint(equalToConstant: 24),
        ])
        
        //비디오 슬라이더
        NSLayoutConstraint.activate([
            videoSlider.rightAnchor.constraint(equalTo: videoLengthLabel.leftAnchor, constant: -8),
            videoSlider.bottomAnchor.constraint(equalTo: progressView.bottomAnchor, constant: -8),
            videoSlider.leftAnchor.constraint(equalTo: currentTimeLabel.rightAnchor),
            videoSlider.heightAnchor.constraint(equalToConstant: 24),
        ])
        
        //제목
        NSLayoutConstraint.activate([
            titlelabel.leftAnchor.constraint(equalTo: playerView.leftAnchor, constant: 20),
            titlelabel.bottomAnchor.constraint(equalTo: playerView.bottomAnchor, constant: 90),
        ])
        
        //부제목
        NSLayoutConstraint.activate([
            subtitlelabel.leftAnchor.constraint(equalTo: playerView.leftAnchor, constant: 20),
            subtitlelabel.bottomAnchor.constraint(equalTo: titlelabel.bottomAnchor, constant: 28),
        ])
        
        //좋아요 수
        NSLayoutConstraint.activate([
            likenumber.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -20),
            likenumber.bottomAnchor.constraint(equalTo: likebutton.bottomAnchor),
        ])
        
        //좋아요 버튼
        NSLayoutConstraint.activate([
            likebutton.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -50),
            likebutton.bottomAnchor.constraint(equalTo: subtitlelabel.bottomAnchor),
            likebutton.widthAnchor.constraint(equalToConstant: 20),
            likebutton.heightAnchor.constraint(equalToConstant: 20),
        ])
        
        //스킵버튼
        NSLayoutConstraint.activate([
            skipbutton.rightAnchor.constraint(equalTo: view.rightAnchor),
            skipbutton.topAnchor.constraint(equalTo: progressView.topAnchor, constant: -40),
            skipbutton.widthAnchor.constraint(equalToConstant: 120),
            skipbutton.heightAnchor.constraint(equalToConstant: 40),
        ])
        
        
        //
        playerView.layer.addSublayer(playerLayer)
        playerLayer.player = player
        
        
        //비디오 로딩이 끝나면 일시정지 버튼으로 변경됨
        player.addObserver(self, forKeyPath: "currentItem.loadedTimeRanges", options: .new, context: nil)
        
        //
        let interval = CMTime(value: 1, timescale: 2)
        player.addPeriodicTimeObserver(forInterval: interval, queue: DispatchQueue.main, using: { (progressTime) in
            let seconds = CMTimeGetSeconds(progressTime)
            let secondsString = String(format: "%02d", Int(seconds) % 60)
            let minutesString = String(format: "%02d", Int(seconds) / 60)
            self.currentTimeLabel.text = "\(minutesString):\(secondsString)"
            
            NotificationCenter.default.addObserver(self, selector: #selector(DancePlayViewController.didfinishPlaying(note:)), name: NSNotification.Name.AVPlayerItemDidPlayToEndTime, object: self.player.currentItem)
            
            
            
            if let duration = self.player.currentItem?.duration {
                let durationSeconds = CMTimeGetSeconds(duration)
                
                //print("value of duratioinSeconds: \(seconds))")
                //print("value of Seconds: \(durationSeconds)")
                
                self.videoSlider.value = Float(seconds / durationSeconds)
            
            }
        })
    }
    
    
    //
    override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
        //
        if keyPath == "currentItem.loadedTimeRanges" {
            activityIndicatorView.stopAnimating()
            playerView.backgroundColor = .clear
            pausePlayButton.isHidden = false
            isPlaying = true
            
            //
            //if let duration = player.currentItem?.duration {
                //let seconds = Int(CMTimeGetSeconds(duration))
                //let secondsText = String(format: "%02d", seconds % 60)
                //let minutesText = String(format: "%02d", seconds / 60)
                //videoLengthLabel.text = "\(minutesText):\(secondsText)"
            //}
        }
    }
        
    //
    @objc func handleSliderChange() {
        print(videoSlider.value)
        
        if let duration = player.currentItem?.duration {
            let totalSeconds = CMTimeGetSeconds(duration)
            let value = Float64(videoSlider.value) * totalSeconds
            let seekTime = CMTime(value: Int64(value), timescale: 1)
            player.seek(to: seekTime, completionHandler: { (completedSeek) in
                
            })
        }
    }
    
    //
    @objc func handlePause() {
        
        //
        if isPlaying {
            player.pause()
            pausePlayButton.setImage(UIImage(named: "play"), for: .normal)

        } else {
            player.play()
            pausePlayButton.setImage(UIImage(named: "pause"), for: .normal)
        }
        isPlaying = !isPlaying
    }
    
    //춤추기 버튼 클릭 이벤트
    @objc func pressed(sender: UIButton) {
        let dialog = UIAlertController(title: "춤추기 버튼이 눌렸습니다.", message: "내용", preferredStyle: .alert)
        let action = UIAlertAction(title: "확인", style: UIAlertAction.Style.default)
        dialog.addAction(action)
        self.present(dialog, animated: true, completion: nil)
        
    }
    
    //좋아요 버튼 클릭 이벤트
    @objc func pressed2(sender: UIButton) {
        let dialog = UIAlertController(title: "좋아요 버튼이 눌렸습니다.", message: "내용", preferredStyle: .alert)
        let action = UIAlertAction(title: "확인", style: UIAlertAction.Style.default)
        dialog.addAction(action)
        self.present(dialog, animated: true, completion: nil)
    }
    
    //버튼클릭시 숫자가 1씩 증가
    //@objc func buttonAction(sender: UIButton) {
        //counter += 1
        //print(counter)
    //}
    
    //
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        playerLayer.frame = playerView.bounds
    }
    
    //광고 Skip 버튼 이벤트
    @objc func update() {
        
        //counter 값이 0보다 크면
        if (counter > 0) {
            counter -= 1
            skipbutton.isHidden = false
            skipbutton.setTitle("\(counter)" + "초 후 Skip", for: .normal)
            print("\(counter)" + "초 후 Skip")
            
        //counter 값이 0이면
        } else if (counter == 0) {
            skipbutton.setTitle("광고 Skip하기", for: .normal)
            
        }
    }
    
    //Skip 버튼 클릭 이벤트
    @objc func skipbuttonAction(sender: UIButton) {
        print("button click")
    }
    
    //비디오 재생이 끝났을때 발생되는 이벤트
    @objc func didfinishPlaying(note : NSNotification)  {
        skipbutton.isHidden = true
        playerLayer.backgroundColor = UIColor.black.cgColor
    }
    
    //
    deinit {
        NotificationCenter.default.removeObserver(self)
    }
    
    //
//    func getFullPath(fileName: String!) -> URL! {
//        guard let url = URL(string: "http://download.appboomclap.co.kr/"), let fileName = fileName else {
//            return nil
//        }
//        return URL(string: "\(url)\(fileName)")!
//    }
}
