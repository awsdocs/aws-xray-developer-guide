# Deep linking to the X\-Ray console<a name="scorekeep-deeplinks"></a>

The session and game pages of the Scorekeep web app use deep linking to link to filtered trace lists and service maps\.

**Example [https://github.com/awslabs/eb-java-scorekeep/tree/xray/public/game.html](https://github.com/awslabs/eb-java-scorekeep/tree/xray/public/game.html) – Deep links**  

```
<div id="xray-link">
  <p><a href="https://console.aws.amazon.com/xray/home#/traces?filter=http.url%20CONTAINS%20%22{{ gameid }}%22&timeRange=PT1H" target="blank">View traces for this game</a></p>
  <p><a href="https://console.aws.amazon.com/xray/home#/service-map&timeRange=PT1H" target="blank">View service map</a></p>
</div>
```

See [Deep linking](xray-console-deeplinks.md) for details on how to construct deep links\.