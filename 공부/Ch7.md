# 스프링MVC -웹페이지만들기
----

`@RequiredArgsConstructor`

final이 붙은 멤버변수만 사용해서 생성자를 자동으로 만들어준다.

```ruby
public BasicItemController(ItemRepository itemRepository) {
   this.itemRepository = itemRepository;
}
```
* 이렇게 생성자가 딱 1개만 있으면 스프링이 해당 생성자에 `@Autowired`로 의존관계를 주입해준다.
* 따라서 **final 키워드를 빼면 안된다!**,그러면 `ItemRepository` 의존관계 주입이 안된다.



테스트용 데이터 추가하는 법
* 테스트용 데이터가 없으면 회원 목록이 정상 작동하는지 어려움
* `@PostConstruct`:해당 빈의 의존관계가 모두 주입되고 나면 초기화 용도로 호출된다.

