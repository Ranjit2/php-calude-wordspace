# Template: Repository

```php
<?php declare(strict_types=1);

namespace App\Repositories;

use App\Models\{Name};
use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Eloquent\Collection;

final class {Name}Repository {
    public function findById(int $id): ?{Name} {
        return {Name}::find($id);
    }

    public function findOrFail(int $id): {Name} {
        return {Name}::findOrFail($id);
    }

    public function paginate(int $perPage = 15, array $filters = []): LengthAwarePaginator {
        $query = {Name}::with([/* relationships */])->latest();

        if (!empty($filters['status'])) {
            $query->where('status', $filters['status']);
        }

        return $query->paginate($perPage);
    }

    public function create(array $data): {Name} {
        return {Name}::create($data);
    }

    public function update({Name} $model, array $data): {Name} {
        $model->update($data);
        return $model->fresh();
    }

    public function delete({Name} $model): bool {
        return $model->delete();
    }
}
```
