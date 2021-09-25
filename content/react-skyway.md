---
title: 'React+Skywayでビデオ通話機能'
date: 2021-09-25
tags: ['React', 'Skyway']
---

今回実装する機能は以下の通りです。

- 発信&受信
- ビデオ on/off
- 音声 on/off
- カメラ in/out

## 初期設定

① こちらから Skyway に登録します。  
https://webrtc.ecl.ntt.com/

② 登録後、アプリケーションを作成+利用可能ドメインを設定し(localhost 等)、API キーをメモします。

③ プロジェクトにパッケージをインストールします。

```shell
yarn add skyway-js
```

## 本題

TypeScript で書いてます。

```tsx {fn="skyway-sample.tsx"}
import { useEffect, useState, useRef } from 'react';
import Peer, { MediaConnection } from 'skyway-js';
let local_stream: MediaStream; // 自分のストリーミング情報を保存しておく変数

// まずはpeer設定をする
const CallArea = () => {
  const [peer, set_peer] = useState<Peer>();
  const [my_peer_id, set_my_peer_id] = useState<string>();

  useEffect(() => {
    const peer = new Peer({
      key: 'your api key', // ここにメモしたAPIキーをセット
    });
    set_peer(peer);
    peer.once('open', (id) => {
      set_my_peer_id(id);
    });
  }, []);

  if (peer === undefined || my_peer_id === undefined) return <></>; // peer設定&自分のpeer_id取得を待つ
  return <CallAreaMain peer={peer} my_peer_id={my_peer_id} />;
};

const CallAreaMain: React.FC<{
  peer: Peer;
  my_peer_id: string;
}> = ({ peer, my_peer_id }) => {
  // 自分のビデオを入れる要素 相手のビデオを入れる要素
  const my_video = useRef<HTMLVideoElement>(null);
  const partner_video = useRef<HTMLVideoElement>(null);

  // 自分の映像を取得
  useEffect(() => {
    navigator.mediaDevices
      .getUserMedia({
        video: true,
        audio: true,
      })
      .then((stream) => {
        if (my_video.current !== null) {
          my_video.current.srcObject = stream;
        }
        local_stream = stream;
      })
      .catch((error) => {
        console.error(error);
      });
  }, []);

  // 相手との接続
  const [media_connection, set_media_connection] = useState<MediaConnection>();

  // 相手との接続をstateに保持 & 相手のストリーミング情報をvideoタグにセットする共通関数
  const set_partner_video = (mc: MediaConnection) => {
    set_media_connection(mc);
    mc.on('stream', (stream) => {
      if (partner_video.current !== null) {
        partner_video.current.srcObject = stream;
      }
    });
  };

  // 相手から受信
  peer.on('call', (mc) => {
    mc.answer(local_stream);
    set_partner_video(mc);
  });

  // 相手に発信
  const [partner_peer_id, set_partner_peer_id] = useState('');
  const call = () => {
    const mc = peer.call(partner_peer_id, local_stream);
    set_partner_video(mc);
  };

  // 音声 ON/OFF
  const [audio_on, set_audio_on] = useState(true);
  const toggle_audio = () => {
    local_stream.getAudioTracks()[0].enabled = !audio_on;
    set_audio_on(!audio_on);
  };

  // カメラ ON/OFF
  const [video_on, set_video_on] = useState(true);
  const toggle_video = () => {
    local_stream.getVideoTracks()[0].enabled = !video_on;
    set_video_on(!video_on);
  };

  // カメラ in/out
  const [camera_type, set_camera_type] = useState<'in' | 'out'>('in');
  const change_camera_type = (change_to: 'in' | 'out') => {
    if (video_on === false) return; // ビデオをオンにしてください

    // 既に開いているカメラ/audioを閉じる (重要。これをしないと新しいカメラ/audioが開けない)
    local_stream.getAudioTracks()[0].stop();
    local_stream.getVideoTracks()[0].stop();

    const facingMode: ConstrainDOMString =
      change_to === 'in' ? 'user' : { exact: 'environment' };
    navigator.mediaDevices
      .getUserMedia({
        video: { facingMode },
        audio: true,
      })
      .then((stream) => {
        // ミュートしている場合、デフォルト値をミュートにする
        if (audio_on === false) stream.getAudioTracks()[0].enabled = false;

        // カメラタイプ(in/out)を変更
        set_camera_type(change_to);
        if (my_video.current !== null) {
          my_video.current.srcObject = stream;
        }
        local_stream = stream;
        media_connection?.replaceStream(stream); // 相手に新しいメディア情報を送信
      })
      .catch((error) => {
        // デバイスにアウトカメラがない場合のエラー (PCなど)
        if (error.name === 'OverconstrainedError') {
          console.error('お使いの端末ではこの機能をお使いいただけません');
          change_camera_type('in'); // インカメラで接続し直し
        }
      });
  };

  return (
    <div>
      {/* 相手に自分のpeer_idを入力してもらう場合 */}
      <p>相手に伝えるID：{my_peer_id}</p>

      {/* 自分が相手のpeer_idを入力し発信する場合 */}
      <div>
        <input
          type="text"
          onChange={(e) => set_partner_peer_id(e.target.value)}
        />
        <button onClick={call}>相手に発信</button>
      </div>

      {/* 相手のビデオ */}
      <video ref={partner_video} autoPlay playsInline />

      {/* 自分のビデオ (自分の音声は聞こえなくて良いので、muted属性をつける) */}
      <video ref={my_video} autoPlay muted playsInline />

      {/* 音声 ON/OFF */}
      <button onClick={toggle_audio}>
        {audio_on === true ? '○音声がオンです' : '✖︎音声がオフです'}
      </button>

      {/* ビデオ ON/OFF */}
      <button onClick={toggle_video}>
        {video_on === true ? '○ビデオがオンです' : '✖︎ビデオがオフです'}
      </button>

      {/* カメラ in/out */}
      <button
        onClick={() => change_camera_type(camera_type === 'in' ? 'out' : 'in')}
      >
        {camera_type === 'in' ? '今インカメラです' : '今アウトカメラです'}
      </button>
    </div>
  );
};

export default CallArea;
```
