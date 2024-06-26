[先週のコラム][taproot series vaults]では、Antoine Poinsotが、
[Vault][topic vaults]スタイルのコインバックアップやセキュリティスキームを、
Taprootによって、よりプライベートで手数料を効率化する方法を説明しました。
今週のコラムでは、Taprootに変更することで改善される他のいくつかのバックアップやセキュリティスキームについて見ていきます。

- **シンプルな2-of-3:** 先週[述べたように][threshold signing]、
  [マルチシグ][topic multisignature]とscriptpathによる支払いを組み合わせて、
  2-of-3の支払いポリシーを簡単に作ることができます。通常ケースはシングルシグの支払いと同じくらい効率的なオンチェーンで、
  現在のP2SHやP2WSHのマルチシグよりもはるかにプライベートなものです。
  例外ケースでも、かなり効率的でプライベートです。このように、Taprootは、
  単一署名者のウォレットから複数の署名者のポリシーにセキュリティをアップグレードするのに適しています。

  今後の[閾値署名][topic threshold signature]の技術により、
  2-of-3や他のk-of-nのケースがさらに改善されることを期待しています。

- **マルチシグのデグレード:** [Optech Taproot Workshop][taproot workshop]の演習問題の1つでは、
  3つの鍵ではいつでも支払いが可能、もしくは3日後に元の鍵の内2つで支払いが可能、
  10日後であれば元の鍵の内1つで支払いが可能なTaprootのScriptを作成する実験をすることができます。
  （この実験では、バックアップの鍵も使用しますが、それは次のポイントで別途取り上げます。）
  このように、時間パラメーターと鍵の設定を調整することで、柔軟かつ強力なバックアップを構築することができます。

  例えば、普段はノートパソコンや携帯電話、ハードウェア署名デバイスを組み合わせて支払いをできるとします。
  そのうちの1つが使えなくなった場合、1ヶ月待てば残りの2つのデバイスを使って支払いができます。
  2つのデバイスが使えなくなった場合は、6ヶ月後に1つのデバイスだけで支払いができます。

  3つのデバイスをすべて使用する通常のケースでは、あなたのオンチェーンScriptは最大限に効率的でプライベートなものになります。
  その他のケースでは、少し効率は低下しますが、それでもかなりプライベートである可能性があります
  （あなたのScriptとそのツリーの深さは、他の多くのコントラクトで使用されているScriptと深さと似ています）。

- **バックアップのためのソーシャルリカバリーとセキュリティ:** 上記の例は、
  デバイスの1つが攻撃者に盗まれた場合の保護に優れていますが、2つのデバイスが盗まれた場合はどうなるでしょう？
  また、ウォレットを頻繁に使用する場合、デバイスを失った後に再び支払いを開始できるようになるまで本当に1ヶ月待ちたいと思いますか？

  Taprootを使うと、バックアップにソーシャル要素を簡単かつ安価、プライベートに追加できます。
  前の例のScriptに加えて、2つのデバイスに加えて友人または家族からの署名があればビットコインをすぐに使用できるようにすることもできます。
  または、鍵の1つと信頼できる5人の署名があればすぐに使用できます。
  （これと同様の非ソーシャルバージョンは、安全な場所に保管してある追加のデバイスやシードフレーズを使用するだけです。）

- **相続のための時間的および社会的閾値の組み合わせ:** 上記の技術を組み合わせることで、
  あなたが突然死亡したりだめになってしまった場合に、誰かまたは複数人の人があなたの資金を回収できるようにすることができます。
  例えば、ビットコインを6ヶ月動かさなかった場合、
  あなたの弁護士と最も信頼できる5人の親族のうち任意の3人にコインの使用を許可することができます。
  普段からビットコインを6ヶ月ごとに移動させていれば、この相続の準備は、
  あなたが生きている限りオンチェーンコストがかからず、外部からは完全に非公開になります。
  あなたの死後、弁護士や家族があなたのウォレットの拡張公開鍵（xpub）を知ることができる確実な方法を用意しておけば、
  トランザクションを弁護士や家族に秘密にしたままにすることも可能です。

  なお相続人があなたのビットコインを使用できるようにすることは、
  そのビットコインを合法的に使用できることを意味するわけではないことに注意してください。
  ビットコインの相続を計画されている方は、Pamela Morgan著
  *Cryptoasset Inheritance Planning*（[物理的な書籍およびDRM付き電子書籍][cip amazon]、
  または[DRMフリーの電子書籍][cip aantonop]）をお読みになり、
  その情報をもとに、地元の相続計画の専門家に詳細を相談されることをお勧めします。

- **侵害の検出:** Taprootの発明以前に[提案された][tree signatures]アイディアは、
  デバイスが侵害されたことを検出する方法として、いくつかの量のビットコインを管理する鍵を、
  あなたが気をつけているデバイスのすべてに配置するというものです。ビットコインの量が十分多ければ、
  攻撃者はおそらく、不正アクセスを長期的な攻撃に利用して全体的な被害が広がるのを待つのではなく、
  目先の利益のためにそのコインを使用するでしょう。

  このアプローチの問題は、提供するビットコインの量を攻撃者を誘うのに十分な大きさにしたいものの、
  大量のビットコインをすべてのデバイスに配置したくはなく、どちらかというと大きな報奨金を1つだけ提供したいという点です。
  しかし、すべてのデバイスに同じ鍵を配置すると、攻撃者がビットコインを使用するトランザクションからは、
  どのデバイスが侵害されたかはわかりません。
  Taprootでは、すべてのデバイスに異なるscriptpathの異なる鍵を簡単に配置することができます。
  これらの鍵のいずれかが、そのアドレスで管理されているすべての資金を使用することができますが、
  どのデバイスが侵害されたか一意に特定することができます。

[taproot series vaults]: /ja/preparing-for-taproot/#vaultとtaproot
[cip amazon]: https://amazon.com/Cryptoasset-Inheritance-Planning-Simple-Owners/dp/1947910116
[cip aantonop]: https://aantonop.com/product/cryptoasset-inheritance-planning-a-simple-guide-for-owners/
[tree signatures]: https://blockstream.com/2015/08/24/en-treesignatures/#h.2lysjsnoo7jd
[threshold signing]: /ja/preparing-for-taproot/#閾値署名
[taproot workshop]: https://github.com/bitcoinops/taproot-workshop
