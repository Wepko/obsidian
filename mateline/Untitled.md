use OpenApi\Attributes as OA;

#[OA\Schema(
    schema: 'PrintBlankField',
    description: 'Элемент формы для печатного бланка',
    required: ['name', 'title']
)]
class PrintBlankField
{
    #[OA\Property(
        property: 'name',
        type: 'string',
        example: 'username',
        description: 'Системное имя поля'
    )]
    public string $name;

    #[OA\Property(
        property: 'title',
        type: 'string',
        example: 'Полное имя',
        description: 'Человекочитаемый заголовок поля'
    )]
    public string $title;

    #[OA\Property(
        property: 'value',
        oneOf: [
            new OA\Schema(type: 'string'),
            new OA\Schema(type: 'integer'),
            new OA\Schema(type: 'boolean'),
            new OA\Schema(type: 'null')
        ],
        example: ' ',
        description: 'Текущее значение поля',
        nullable: true
    )]
    public mixed $value;

    #[OA\Property(
        property: 'required',
        type: 'boolean',
        example: true,
        description: 'Обязательность заполнения поля',
        default: false
    )]
    public bool $required = false;

    #[OA\Property(
        property: 'disabled',
        type: 'boolean',
        example: true,
        description: 'Доступно ли поле для редактирования',
        default: false